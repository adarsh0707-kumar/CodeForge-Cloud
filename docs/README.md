# CodeForge Cloud

## Documentation

**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Documentation Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. Documentation Overview

This directory contains the complete technical documentation for **CodeForge Cloud**.

CodeForge Cloud is a cloud-native collaborative development platform that provides:

* browser-based code editing;
* project and file management;
* real-time collaboration;
* asynchronous code execution;
* secure sandboxed compilation;
* execution history;
* resource-controlled workloads;
* distributed service communication;
* observability;
* security controls;
* automated testing;
* future cloud-scale deployment.

The documentation is organized so that a developer, architect, security engineer, tester, or contributor can understand the platform from requirements through implementation and deployment.

---

# 2. Documentation Structure

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

---

# 3. Document Index

| #      | Document             | Purpose                                                                                |
| ------ | -------------------- | -------------------------------------------------------------------------------------- |
| 01     | Product Requirements | Defines product vision, users, features, requirements, and MVP                         |
| 02     | Architecture         | Defines system architecture, services, data flow, and boundaries                       |
| 03     | Data Model           | Defines PostgreSQL schema, Redis model, entities, constraints, and lifecycle           |
| 04     | API Reference        | Defines public REST, WebSocket, and internal service contracts                         |
| 05     | Roadmap & Phases     | Defines development phases, milestones, risks, and release strategy                    |
| 06     | Development Guide    | Defines local development, workflows, debugging, and contribution practices            |
| 07     | Security             | Defines threat model, sandbox security, isolation, and security controls               |
| 08     | Gap Analysis         | Identifies missing capabilities, architectural gaps, risks, and remediation priorities |
| 09     | Testing Strategy     | Defines unit, integration, E2E, security, load, and release testing                    |
| 10     | Glossary             | Defines canonical CodeForge terminology                                                |
| README | Documentation Index  | Provides navigation and overview of all documentation                                  |

---

# 4. Recommended Reading Order

New developers should read the documents in this order:

```text
01 Product Requirements
        ↓
02 Architecture
        ↓
03 Data Model
        ↓
04 API Reference
        ↓
05 Roadmap & Phases
        ↓
06 Development Guide
        ↓
07 Security
        ↓
08 Gap Analysis
        ↓
09 Testing Strategy
        ↓
10 Glossary
```

This order moves from:

```text
WHY
 ↓
WHAT
 ↓
HOW
 ↓
DATA
 ↓
INTERFACES
 ↓
IMPLEMENTATION
 ↓
SECURITY
 ↓
QUALITY
 ↓
TERMINOLOGY
```

---

# 5. Document 01 — Product Requirements

**File:**

```text
01-product-requirements.md
```

This document defines what CodeForge Cloud is expected to accomplish.

It covers:

* product vision;
* problem statement;
* target users;
* user journeys;
* functional requirements;
* non-functional requirements;
* MVP scope;
* execution requirements;
* collaboration requirements;
* security requirements;
* performance requirements;
* observability requirements;
* future capabilities.

### Read this document when:

* understanding product goals;
* defining new features;
* deciding MVP scope;
* evaluating whether a feature belongs in the platform.

---

# 6. Document 02 — Architecture

**File:**

```text
02-architecture.md
```

This document defines how CodeForge Cloud is structured.

Major architectural components include:

```text
React Web IDE
      ↓
Nginx
      ↓
Node.js Gateway
      ↓
Python Evaluator
      ↓
Redis Queue
      ↓
Worker
      ↓
C++ Sandbox
      ↓
Docker Isolation
```

Supporting infrastructure includes:

```text
PostgreSQL
Redis
gRPC
Protocol Buffers
WebSocket
Docker
Nginx
```

### Read this document when:

* implementing services;
* changing architecture;
* adding infrastructure;
* reviewing service boundaries;
* designing execution flows.

---

# 7. Document 03 — Data Model

**File:**

```text
03-data-model.md
```

This document defines persistent and transient application data.

Core entities include:

```text
User
Project
ProjectMember
File
ExecutionJob
ExecutionResult
ExecutionSnapshot
Session
AuditEvent
ResourcePolicy
```

PostgreSQL is the durable source of truth.

Redis is used for:

* queues;
* cache;
* pub/sub;
* presence;
* rate limiting;
* transient state.

A key architectural rule is:

> An execution runs against an immutable snapshot, never against mutable live project state.

### Read this document when:

* creating database tables;
* writing migrations;
* modifying relationships;
* implementing execution persistence;
* designing Redis keys.

---

# 8. Document 04 — API Reference

**File:**

```text
04-api-reference.md
```

This document defines the external and internal service contracts.

Public API:

```text
/api/v1
```

Major API groups:

```text
Authentication
Projects
Project Members
Files
Executions
Snapshots
Collaboration
```

Example:

```http
POST /api/v1/executions
```

Execution submission is asynchronous and returns:

```http
202 Accepted
```

Internal communication uses:

```text
gRPC + Protocol Buffers
```

Real-time communication uses:

```text
WebSocket
```

### Read this document when:

* implementing frontend API calls;
* implementing Gateway endpoints;
* integrating services;
* changing API contracts;
* designing WebSocket events.

---

# 9. Document 05 — Roadmap & Phases

**File:**

```text
05-roadmap-and-phases.md
```

This document defines the implementation sequence.

The roadmap progresses through:

```text
Phase 0  → Foundation
Phase 1  → Data Layer
Phase 2  → Authentication
Phase 3  → Projects & Files
Phase 4  → Execution API
Phase 5  → Python Evaluator
Phase 6  → C++ Sandbox
Phase 7  → End-to-End Execution
Phase 8  → Web IDE
Phase 9  → Collaboration
Phase 10 → Observability
Phase 11 → Security Hardening
Phase 12 → Performance
Phase 13 → Production Deployment
Phase 14 → Advanced Platform
```

### Read this document when:

* planning implementation;
* creating issues;
* deciding sprint scope;
* determining MVP boundaries;
* planning releases.

---

# 10. Document 06 — Development Guide

**File:**

```text
06-development-guide.md
```

This is the primary developer onboarding document.

It covers:

* environment setup;
* Docker;
* PostgreSQL;
* Redis;
* frontend development;
* Gateway development;
* evaluator development;
* sandbox development;
* gRPC;
* WebSockets;
* testing;
* debugging;
* Git workflow;
* code quality;
* security practices;
* local development;
* troubleshooting.

### Read this document when:

* setting up the project;
* starting development;
* debugging services;
* contributing code;
* running the complete platform locally.

---

# 11. Document 07 — Security

**File:**

```text
07-security.md
```

This document defines the security architecture.

The most important security assumption is:

> User source code is hostile input.

Security controls include:

```text
Authentication
Authorization
Input Validation
Resource Limits
Container Isolation
Linux Namespaces
cgroups
seccomp
Capability Restrictions
Filesystem Isolation
Network Isolation
Timeouts
Output Limits
Audit Logging
Rate Limiting
```

The sandbox must never provide user workloads with:

```text
Docker Socket
Host Filesystem
PostgreSQL Access
Redis Access
Unauthorized Network Access
Host Privileges
```

### Read this document when:

* modifying sandbox behavior;
* handling untrusted input;
* changing authentication;
* changing authorization;
* adding infrastructure;
* reviewing security-sensitive code.

---

# 12. Document 08 — Gap Analysis

**File:**

```text
08-gap-analysis.md
```

This document identifies areas where the platform requires additional implementation, validation, hardening, or production readiness.

Typical gap categories include:

```text
Architecture Gaps
Implementation Gaps
Security Gaps
Testing Gaps
Observability Gaps
Scalability Gaps
Deployment Gaps
Documentation Gaps
Operational Gaps
```

### Read this document when:

* deciding what remains unfinished;
* planning future work;
* reviewing MVP readiness;
* preparing production deployment.

---

# 13. Document 09 — Testing Strategy

**File:**

```text
09-testing-strategy.md
```

This document defines how CodeForge Cloud is validated.

Testing covers:

```text
Unit
Integration
API
Database
Redis
gRPC
WebSocket
E2E
Security
Sandbox
Load
Stress
Soak
Regression
CI
Production Smoke
```

Security testing is especially important because user source code is treated as hostile.

Critical tests include:

* sandbox escape attempts;
* resource exhaustion;
* process explosion;
* output flooding;
* filesystem traversal;
* network access;
* Docker socket access;
* privilege escalation;
* cross-project access;
* authentication abuse;
* authorization bypass.

### Read this document when:

* adding features;
* writing tests;
* preparing pull requests;
* preparing releases;
* modifying security-sensitive code.

---

# 14. Document 10 — Glossary

**File:**

```text
10-glossary.md
```

This document defines canonical terminology.

Important terms include:

```text
Gateway
Evaluator
Worker
Sandbox
Execution
Execution Job
Execution Snapshot
Execution Result
Resource Policy
Project
Project Member
CRDT
WebSocket
gRPC
cgroup
seccomp
Namespace
Idempotency
Optimistic Concurrency
Tenant Isolation
Defense in Depth
```

### Read this document when:

* a term is unclear;
* introducing new terminology;
* writing documentation;
* reviewing architecture;
* naming components.

---

# 15. Core Architecture

The complete platform can be summarized as:

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
                       REST/API │       │ WebSocket
                                │       │
                                ▼       ▼
                       ┌────────────┐  Collaboration
                       │ PostgreSQL │
                       └────────────┘

                                │
                              gRPC
                                │
                                ▼
                       ┌─────────────────┐
                       │ Python Evaluator│
                       └────────┬────────┘
                                │
                                ▼
                          ┌───────────┐
                          │   Redis   │
                          │ Queue/Bus │
                          └─────┬─────┘
                                │
                              Worker
                                │
                                ▼
                       ┌─────────────────┐
                       │  C++ Sandbox   │
                       └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │ Docker Runtime  │
                       │ Namespaces      │
                       │ cgroups         │
                       │ seccomp         │
                       │ Capabilities    │
                       │ Network Isolation│
                       └────────┬────────┘
                                │
                                ▼
                         User Program
                                │
                                ▼
                       stdout / stderr
                                │
                                ▼
                         Execution Result
```

---

# 16. Core Execution Flow

The standard execution lifecycle is:

```text
User clicks Run
      ↓
Gateway receives request
      ↓
Authentication
      ↓
Authorization
      ↓
Validate project/file
      ↓
Capture immutable snapshot
      ↓
Persist execution job
      ↓
Queue execution
      ↓
Evaluator receives job
      ↓
Worker processes job
      ↓
Sandbox created
      ↓
Snapshot materialized
      ↓
Resource limits applied
      ↓
Compile
      ↓
Execute
      ↓
Capture stdout/stderr
      ↓
Capture resource usage
      ↓
Persist result
      ↓
Publish execution status
      ↓
Web IDE displays result
```

---

# 17. Core Security Model

The platform follows a defense-in-depth security model:

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
                     Snapshot
                           │
                           ▼
                       Sandbox
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Namespace      cgroup      seccomp
              │            │            │
              └────────────┼────────────┘
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

# 18. Core Data Flow

```text
Live Project
     │
     ▼
Snapshot Capture
     │
     ▼
Immutable Snapshot
     │
     ▼
Execution Job
     │
     ▼
Redis Queue
     │
     ▼
Worker
     │
     ▼
Sandbox
     │
     ▼
Compile + Execute
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
Web IDE
```

---

# 19. Repository Structure

The documentation describes the following repository architecture:

```text
Online-Code-Compiler/
│
├── docs/
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

# 20. Technology Stack

| Layer             | Technology              |
| ----------------- | ----------------------- |
| Frontend          | React                   |
| Frontend Language | TypeScript              |
| Build Tool        | Vite                    |
| Code Editor       | Monaco Editor           |
| UI                | Tailwind CSS            |
| Icons             | Lucide                  |
| Gateway           | Node.js                 |
| Gateway Language  | TypeScript              |
| API               | REST                    |
| Real-Time         | WebSocket / Socket.IO   |
| Evaluator         | Python                  |
| Evaluator API     | FastAPI                 |
| Internal RPC      | gRPC                    |
| Serialization     | Protocol Buffers        |
| Sandbox           | C++                     |
| Isolation         | Docker + Linux Security |
| Database          | PostgreSQL              |
| Queue             | Redis                   |
| Reverse Proxy     | Nginx                   |
| Local Deployment  | Docker Compose          |
| Future Deployment | Kubernetes              |

---

# 21. Architectural Principles

CodeForge Cloud follows these core principles:

### 1. Security by Design

Security is part of the architecture rather than an afterthought.

### 2. Least Privilege

Services receive only the permissions they require.

### 3. Separation of Concerns

Each service has a clearly defined responsibility.

### 4. Contract First

Public and internal interfaces are explicitly defined.

### 5. Disposable Execution

Execution environments are temporary.

### 6. Immutable Execution Input

Executions use immutable snapshots.

### 7. Observable by Default

Important operations produce logs, metrics, and traces.

### 8. Fail Closed

Security uncertainty results in denial.

### 9. Defense in Depth

Multiple security layers protect the execution boundary.

### 10. Durable Source of Truth

PostgreSQL stores authoritative persistent state.

---

# 22. Documentation Ownership

| Area         | Primary Document |
| ------------ | ---------------- |
| Product      | 01               |
| Architecture | 02               |
| Database     | 03               |
| APIs         | 04               |
| Planning     | 05               |
| Development  | 06               |
| Security     | 07               |
| Gaps         | 08               |
| Testing      | 09               |
| Terminology  | 10               |

When a change affects multiple areas, update all relevant documents.

---

# 23. When to Update Documentation

Documentation must be updated when:

* an API endpoint changes;
* a database schema changes;
* a service is added or removed;
* execution states change;
* security controls change;
* resource limits change;
* deployment architecture changes;
* collaboration behavior changes;
* testing requirements change;
* terminology changes.

Documentation should be updated in the same development change whenever practical.

---

# 24. Documentation Change Workflow

Use the following workflow:

```text
Identify Change
      ↓
Determine Affected Document
      ↓
Update Documentation
      ↓
Review Architecture Consistency
      ↓
Review API/Data/Security Impact
      ↓
Update Tests
      ↓
Commit Changes
```

Example:

```text
New Execution Feature
        │
        ├── Requirements
        ├── Architecture
        ├── API
        ├── Data Model
        ├── Security
        ├── Testing
        └── Glossary
```

---

# 25. Documentation Quality Rules

All CodeForge documentation should:

1. Use consistent terminology.
2. Clearly identify assumptions.
3. Document security boundaries.
4. Document failure behavior.
5. Document API contracts.
6. Document database constraints.
7. Document resource limits.
8. Document important architectural decisions.
9. Avoid undocumented behavior.
10. Keep examples synchronized with implementation.
11. Prefer explicit technical language.
12. Clearly distinguish current functionality from future plans.

---

# 26. Quick Navigation

For a specific task:

### I want to understand the project

Read:

```text
01 → 02
```

### I want to understand the database

Read:

```text
03
```

### I want to build an API

Read:

```text
04
```

### I want to know what to build next

Read:

```text
05
```

### I want to start coding

Read:

```text
06
```

### I want to modify the sandbox

Read:

```text
02 → 07 → 09
```

### I want to perform security work

Read:

```text
07 → 09 → 08
```

### I want to write tests

Read:

```text
09
```

### I do not understand a technical term

Read:

```text
10
```

### I want to know what is incomplete

Read:

```text
08
```

---

# 27. Critical Rules at a Glance

The following rules apply throughout the platform.

```text
1. Browser input is untrusted.

2. User source code is hostile input.

3. Gateway never executes user code.

4. User code never executes directly on the host.

5. Every execution uses an immutable snapshot.

6. Every execution has a resource policy.

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

20. Every important bug should become a regression test.
```

---

# 28. Documentation Completion Matrix

| Document                | Status | Primary Purpose          |
| ----------------------- | ------ | ------------------------ |
| 01 Product Requirements | Draft  | Product definition       |
| 02 Architecture         | Draft  | System architecture      |
| 03 Data Model           | Draft  | Data architecture        |
| 04 API Reference        | Draft  | Service contracts        |
| 05 Roadmap              | Draft  | Development planning     |
| 06 Development Guide    | Draft  | Developer workflow       |
| 07 Security             | Draft  | Security architecture    |
| 08 Gap Analysis         | Draft  | Remaining gaps           |
| 09 Testing Strategy     | Draft  | Quality assurance        |
| 10 Glossary             | Draft  | Terminology              |
| README                  | Draft  | Documentation navigation |

---

# 29. Definition of a Complete Documentation Set

The initial documentation set is considered structurally complete when the following areas are covered:

```text
Product
Architecture
Data
API
Roadmap
Development
Security
Gap Analysis
Testing
Terminology
```

Implementation-specific documentation may be added later.

Potential future documents include:

```text
11-deployment-guide.md
12-operations-runbook.md
13-observability.md
14-disaster-recovery.md
15-performance-benchmarks.md
16-contributing.md
17-changelog.md
18-adr/
```

These are future extensions and are not required for the initial documentation set.

---

# 30. Recommended Future Documentation

As CodeForge Cloud moves toward production, the documentation system can be extended with:

### Deployment Guide

Production deployment instructions for:

* Docker Compose;
* Kubernetes;
* networking;
* secrets;
* TLS;
* infrastructure.

### Operations Runbook

Operational procedures for:

* incidents;
* service failures;
* queue failures;
* database failures;
* Redis failures;
* sandbox incidents.

### Observability Guide

Definitions for:

* metrics;
* dashboards;
* alerts;
* logs;
* traces;
* SLOs.

### Disaster Recovery

Procedures for:

* backup;
* restore;
* database recovery;
* service recovery;
* data-loss scenarios.

### Performance Benchmarks

Measured performance data for:

* API latency;
* compilation;
* execution;
* queue latency;
* WebSocket connections;
* concurrent users.

### Architecture Decision Records

Permanent records of significant architectural decisions.

---

# 31. Documentation Dependency Graph

```text
                  01 Product Requirements
                           │
                           ▼
                    02 Architecture
                     │           │
                     ▼           ▼
               03 Data Model   04 API
                     │           │
                     └─────┬─────┘
                           ▼
                   05 Roadmap
                           │
                           ▼
                  06 Development Guide
                     │           │
                     ▼           ▼
                  07 Security   08 Gap Analysis
                     │           │
                     └─────┬─────┘
                           ▼
                   09 Testing Strategy
                           │
                           ▼
                      10 Glossary
```

---

# 32. Documentation as a Living System

CodeForge documentation is not a static artifact.

The implementation and documentation should evolve together:

```text
Architecture Change
      ↓
Implementation Change
      ↓
Test Change
      ↓
Security Review
      ↓
Documentation Update
      ↓
Code Review
      ↓
Merge
```

A feature is not considered fully complete when only its code has been implemented.

The corresponding documentation, tests, security considerations, and operational behavior must also be addressed.

---

# 33. Final Documentation Principle

The CodeForge Cloud documentation set follows one central principle:

> **The documentation should describe the system that is actually built, while clearly identifying what is planned for the future.**

This prevents:

* undocumented architecture;
* accidental API behavior;
* inconsistent terminology;
* hidden security assumptions;
* missing tests;
* unclear ownership;
* implementation drift.

---

# 34. Final Project Documentation Map

```text
                       CODEFORGE CLOUD
                              │
                              ▼
                  ┌─────────────────────┐
                  │ Product Requirements│
                  │        01           │
                  └──────────┬──────────┘
                             ▼
                  ┌─────────────────────┐
                  │     Architecture    │
                  │        02           │
                  └──────────┬──────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
        ┌────────────────┐      ┌────────────────┐
        │   Data Model   │      │  API Reference │
        │       03       │      │       04       │
        └───────┬────────┘      └───────┬────────┘
                └──────────┬─────────────┘
                           ▼
                  ┌─────────────────────┐
                  │ Roadmap & Phases    │
                  │        05           │
                  └──────────┬──────────┘
                             ▼
                  ┌─────────────────────┐
                  │ Development Guide   │
                  │        06           │
                  └──────────┬──────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
        ┌────────────────┐      ┌────────────────┐
        │    Security    │      │  Gap Analysis  │
        │       07       │      │       08       │
        └───────┬────────┘      └───────┬────────┘
                └──────────┬─────────────┘
                           ▼
                  ┌─────────────────────┐
                  │ Testing Strategy    │
                  │        09           │
                  └──────────┬──────────┘
                             ▼
                  ┌─────────────────────┐
                  │      Glossary       │
                  │        10           │
                  └─────────────────────┘
```

---

# 35. Final Status

**CodeForge Cloud documentation set:**

```text
[01] Product Requirements     ✓
[02] Architecture             ✓
[03] Data Model               ✓
[04] API Reference            ✓
[05] Roadmap & Phases         ✓
[06] Development Guide        ✓
[07] Security                 ✓
[08] Gap Analysis             ✓
[09] Testing Strategy         ✓
[10] Glossary                 ✓
[11] Documentation README     ✓
```

The `docs/` directory now provides a complete initial documentation foundation for CodeForge Cloud.

**Author:** Adarsh Kumar
**Project:** CodeForge Cloud
**Documentation Version:** 1.0
**Status:** Draft

> **Build the system. Document the system. Test the system. Secure the system. Keep all four synchronized.**

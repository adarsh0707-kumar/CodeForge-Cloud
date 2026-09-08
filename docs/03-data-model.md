# CodeForge Cloud

## Data Model & Database Design Document

### Document 03 — Data Model

**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.1
**Status:** Draft
**Author:** Adarsh Kumar

---

# Database Constraints

## 1. Constraint Design Principles

The database must enforce critical integrity rules at the PostgreSQL layer rather than relying exclusively on application code.

The constraint strategy follows these principles:

* Primary keys are mandatory for all durable entities.
* Foreign keys enforce ownership and relationships.
* `NOT NULL` is used wherever a value is required for correctness.
* `UNIQUE` constraints prevent duplicate identities and resources.
* `CHECK` constraints reject invalid enum-like values and impossible states.
* Composite constraints enforce project-scoped uniqueness.
* Database constraints must fail closed.
* Application validation may provide better error messages, but database constraints remain the final integrity boundary.
* Cross-table rules that cannot be represented directly using standard constraints must be enforced transactionally by the owning service.

---

# 2. Users Table Constraints

The `users` table represents platform identities.

### Required constraints

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,

    email VARCHAR(320) NOT NULL,
    username VARCHAR(50) NOT NULL,

    password_hash TEXT NOT NULL,
    display_name VARCHAR(100) NOT NULL,

    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_users_email
        UNIQUE (email),

    CONSTRAINT uq_users_username
        UNIQUE (username),

    CONSTRAINT chk_users_email_nonempty
        CHECK (length(trim(email)) > 0),

    CONSTRAINT chk_users_username_format
        CHECK (username ~ '^[A-Za-z0-9_]{3,50}$'),

    CONSTRAINT chk_users_display_name
        CHECK (length(trim(display_name)) BETWEEN 1 AND 100),

    CONSTRAINT chk_users_status
        CHECK (status IN ('ACTIVE', 'SUSPENDED', 'DELETED'))
);
```

### Rules

* Email must be unique.
* Username must be unique.
* Username cannot contain arbitrary special characters.
* Password hashes cannot be nullable.
* Raw passwords must never be stored.
* Deleted users must not be physically reused as active identities.

---

# 3. Projects Table Constraints

```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY,

    owner_id UUID NOT NULL,

    name VARCHAR(100) NOT NULL,
    description TEXT,

    visibility VARCHAR(20) NOT NULL DEFAULT 'PRIVATE',

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_projects_owner
        FOREIGN KEY (owner_id)
        REFERENCES users(id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_projects_name
        CHECK (length(trim(name)) BETWEEN 1 AND 100),

    CONSTRAINT chk_projects_visibility
        CHECK (visibility IN ('PRIVATE', 'PUBLIC'))
);
```

### Important ownership rule

A project owner cannot be deleted while projects still reference that user.

Therefore:

```text
users
  │
  └── projects.owner_id
          │
          └── ON DELETE RESTRICT
```

This prevents orphaned projects.

---

# 4. Project Members Constraints

```sql
CREATE TABLE project_members (
    project_id UUID NOT NULL,
    user_id UUID NOT NULL,

    role VARCHAR(20) NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    PRIMARY KEY (project_id, user_id),

    CONSTRAINT fk_project_members_project
        FOREIGN KEY (project_id)
        REFERENCES projects(id)
        ON DELETE CASCADE,

    CONSTRAINT fk_project_members_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_project_members_role
        CHECK (role IN ('OWNER', 'EDITOR', 'VIEWER'))
);
```

### Constraints

The composite primary key guarantees:

```text
(project_id, user_id)
```

can exist only once.

A user therefore cannot have duplicate memberships in the same project.

---

# 5. Files Table Constraints

The files table represents both files and directories.

```sql
CREATE TABLE files (
    id UUID PRIMARY KEY,

    project_id UUID NOT NULL,
    parent_id UUID,

    name VARCHAR(255) NOT NULL,
    path TEXT NOT NULL,

    type VARCHAR(20) NOT NULL,

    content TEXT,
    size_bytes BIGINT NOT NULL DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_files_project
        FOREIGN KEY (project_id)
        REFERENCES projects(id)
        ON DELETE CASCADE,

    CONSTRAINT fk_files_parent
        FOREIGN KEY (parent_id)
        REFERENCES files(id)
        ON DELETE CASCADE,

    CONSTRAINT chk_files_name
        CHECK (length(trim(name)) BETWEEN 1 AND 255),

    CONSTRAINT chk_files_type
        CHECK (type IN ('FILE', 'DIRECTORY')),

    CONSTRAINT chk_files_size
        CHECK (size_bytes >= 0),

    CONSTRAINT chk_files_directory_content
        CHECK (
            (type = 'DIRECTORY' AND content IS NULL)
            OR
            (type = 'FILE')
        )
);
```

## Project-scoped path uniqueness

A project cannot contain two resources at the same path.

```sql
CREATE UNIQUE INDEX uq_files_project_path
ON files(project_id, path);
```

Example:

```text
project A
├── src/main.cpp
└── src/utils.cpp
```

is valid.

But:

```text
project A
├── src/main.cpp
└── src/main.cpp
```

must be rejected.

---

# 6. File Parent Integrity

The `parent_id` relationship must remain inside the same project.

The database cannot safely enforce this with a normal single-column foreign key.

Therefore the application/service must validate:

```text
files.parent_id
        ↓
parent.project_id == file.project_id
```

This validation must happen inside the same transaction used to create or move the file.

A future implementation may use a composite foreign key:

```sql
UNIQUE (id, project_id)
```

combined with:

```sql
FOREIGN KEY (parent_id, project_id)
REFERENCES files(id, project_id)
```

This is the preferred stronger database-level design.

---

# 7. Execution Jobs Constraints

Execution jobs are immutable records of requested executions.

```sql
CREATE TABLE execution_jobs (
    id UUID PRIMARY KEY,

    project_id UUID NOT NULL,
    user_id UUID NOT NULL,
    entry_file_id UUID NOT NULL,

    language VARCHAR(30) NOT NULL,
    status VARCHAR(30) NOT NULL DEFAULT 'QUEUED',

    source_snapshot JSONB NOT NULL,
    resource_policy JSONB NOT NULL,

    submitted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,

    CONSTRAINT fk_execution_jobs_project
        FOREIGN KEY (project_id)
        REFERENCES projects(id)
        ON DELETE CASCADE,

    CONSTRAINT fk_execution_jobs_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE RESTRICT,

    CONSTRAINT fk_execution_jobs_entry_file
        FOREIGN KEY (entry_file_id)
        REFERENCES files(id)
        ON DELETE RESTRICT,

    CONSTRAINT chk_execution_jobs_language
        CHECK (length(trim(language)) > 0),

    CONSTRAINT chk_execution_jobs_status
        CHECK (
            status IN (
                'QUEUED',
                'STARTING',
                'COMPILING',
                'RUNNING',
                'COMPLETED',
                'FAILED',
                'TIMEOUT',
                'CANCELLED',
                'RESOURCE_LIMIT'
            )
        ),

    CONSTRAINT chk_execution_jobs_timestamps
        CHECK (
            completed_at IS NULL
            OR started_at IS NULL
            OR completed_at >= started_at
        )
);
```

---

# 8. Entry File Project Integrity

The following relationship is mandatory:

```text
execution_jobs.project_id
        =
files.project_id
```

The execution entry file cannot belong to another project.

This rule must be enforced transactionally.

Preferred database implementation:

```sql
ALTER TABLE files
ADD CONSTRAINT uq_files_id_project
UNIQUE (id, project_id);
```

Then:

```sql
ALTER TABLE execution_jobs
ADD CONSTRAINT fk_execution_entry_file_same_project
FOREIGN KEY (entry_file_id, project_id)
REFERENCES files(id, project_id);
```

This moves an important cross-entity integrity rule directly into PostgreSQL.

---

# 9. Execution Results Constraints

```sql
CREATE TABLE execution_results (
    id UUID PRIMARY KEY,

    job_id UUID NOT NULL UNIQUE,

    stdout TEXT,
    stderr TEXT,

    exit_code INTEGER,
    signal INTEGER,

    compile_time_ms BIGINT,
    execution_time_ms BIGINT,
    memory_bytes BIGINT,

    output_truncated BOOLEAN NOT NULL DEFAULT FALSE,
    resource_violation BOOLEAN NOT NULL DEFAULT FALSE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_execution_results_job
        FOREIGN KEY (job_id)
        REFERENCES execution_jobs(id)
        ON DELETE CASCADE,

    CONSTRAINT chk_execution_results_compile_time
        CHECK (
            compile_time_ms IS NULL
            OR compile_time_ms >= 0
        ),

    CONSTRAINT chk_execution_results_execution_time
        CHECK (
            execution_time_ms IS NULL
            OR execution_time_ms >= 0
        ),

    CONSTRAINT chk_execution_results_memory
        CHECK (
            memory_bytes IS NULL
            OR memory_bytes >= 0
        )
);
```

The `UNIQUE(job_id)` constraint guarantees one final result per execution job.

---

# 10. Sessions Constraints

```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY,

    user_id UUID NOT NULL,

    token_hash TEXT NOT NULL UNIQUE,

    expires_at TIMESTAMPTZ NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    revoked_at TIMESTAMPTZ,

    CONSTRAINT fk_sessions_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE,

    CONSTRAINT chk_sessions_expiry
        CHECK (expires_at > created_at),

    CONSTRAINT chk_sessions_revocation
        CHECK (
            revoked_at IS NULL
            OR revoked_at >= created_at
        )
);
```

Raw session tokens must never be persisted.

Only:

```text
hash(token)
```

is stored.

---

# 11. Audit Events Constraints

```sql
CREATE TABLE audit_events (
    id UUID PRIMARY KEY,

    user_id UUID,
    project_id UUID,

    event_type VARCHAR(100) NOT NULL,

    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_audit_events_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE SET NULL,

    CONSTRAINT fk_audit_events_project
        FOREIGN KEY (project_id)
        REFERENCES projects(id)
        ON DELETE SET NULL,

    CONSTRAINT chk_audit_events_type
        CHECK (length(trim(event_type)) > 0),

    CONSTRAINT chk_audit_events_metadata
        CHECK (jsonb_typeof(metadata) = 'object')
);
```

Audit records should normally remain available even if the associated user or project is removed.

---

# 12. JSONB Constraints

JSONB must not become an unstructured replacement for relational columns.

Use JSONB for:

* execution snapshots
* resource policies
* flexible audit metadata
* future execution metadata

Do not use JSONB for:

* user identity
* project ownership
* membership relationships
* execution status
* file ownership
* security-critical foreign keys

Where practical, JSONB objects should also be validated by the application schema before insertion.

---

# 13. Execution Snapshot Lifecycle

An execution snapshot is the immutable representation of the exact project state used for a specific execution.

The snapshot exists to guarantee:

> The code executed by the sandbox is exactly the code the user submitted for that execution.

The live project filesystem must never be treated as the execution source after job creation.

---

# 14. Snapshot Lifecycle States

The execution snapshot follows this lifecycle:

```text
LIVE PROJECT
     │
     │ Run requested
     ▼
SNAPSHOT CAPTURE
     │
     ▼
SNAPSHOT VALIDATED
     │
     ▼
SNAPSHOT PERSISTED
     │
     ▼
SNAPSHOT QUEUED
     │
     ▼
SNAPSHOT DISPATCHED
     │
     ▼
SANDBOX MATERIALIZED
     │
     ▼
COMPILE
     │
     ▼
EXECUTE
     │
     ▼
RESULT CAPTURED
     │
     ▼
EXECUTION FINALIZED
     │
     ▼
RETENTION / EXPIRATION
     │
     ▼
PURGED
```

---

# 15. Snapshot Capture

When the user clicks **Run**, the Node Gateway must not simply send the current files to the evaluator asynchronously.

Instead:

```text
1. Authenticate request
2. Authorize project access
3. Begin database transaction
4. Read required project files
5. Validate entry file
6. Validate file sizes
7. Capture exact file contents
8. Generate snapshot identifier/hash
9. Create execution_job
10. Commit transaction
11. Publish job to Redis
```

The snapshot therefore represents one exact point in project history.

---

# 16. Snapshot Contents

A snapshot should contain enough information to reproduce the execution.

Example:

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
      "content": "#include <iostream>...",
      "size_bytes": 1024,
      "sha256": "..."
    },
    {
      "path": "src/utils.cpp",
      "content": "...",
      "size_bytes": 512,
      "sha256": "..."
    }
  ],
  "snapshot_sha256": "...",
  "created_at": "2026-09-08T12:00:00Z"
}
```

The snapshot hash provides an integrity identifier for the complete source state.

---

# 17. Snapshot Immutability

Once an execution job reaches:

```text
QUEUED
```

its source snapshot must never change.

The following fields are immutable after job creation:

```text
project_id
user_id
entry_file_id
language
source_snapshot
resource_policy
submitted_at
```

Only execution lifecycle fields may change:

```text
status
started_at
completed_at
```

This distinction is critical.

---

# 18. Why Snapshots Must Be Immutable

Consider this sequence:

```text
10:00:00  User writes version A
10:00:01  User clicks Run
10:00:02  Execution job enters queue
10:00:03  User changes code to version B
10:00:10  Worker starts execution
```

The worker must execute:

```text
Version A
```

not:

```text
Version B
```

Therefore:

```text
Live project state
        ≠
Execution state
```

The snapshot creates a stable boundary between the two.

---

# 19. Snapshot Validation

Before persistence, the evaluator/gateway must validate:

### Project validation

* Project exists.
* User has execution permission.
* Project is not deleted.
* Entry file exists.
* Entry file belongs to the project.

### File validation

* File path is normalized.
* Path traversal is rejected.
* File size is within configured limits.
* Total snapshot size is within project limits.
* Duplicate paths are rejected.
* Unsupported file types are rejected where applicable.

### Language validation

The requested language must be supported.

Example:

```text
cpp
python
javascript
```

Unknown languages must be rejected before sandbox execution.

---

# 20. Snapshot Hashing

Each file should have an individual SHA-256 hash:

```text
sha256(file_content)
```

The complete snapshot should also have a deterministic hash.

For example:

```text
SHA256(
    path + "\0" +
    file_sha256 + "\0" +
    path + "\0" +
    file_sha256 + ...
)
```

Files must be ordered deterministically before calculating the aggregate hash.

This prevents different serialization orders from producing different snapshot identities.

---

# 21. Snapshot Persistence

For the MVP, the snapshot is stored inside:

```text
execution_jobs.source_snapshot
```

using PostgreSQL `JSONB`.

Example:

```text
execution_jobs
└── source_snapshot
    ├── snapshot_id
    ├── files[]
    ├── entry_file
    ├── language
    ├── language_version
    ├── snapshot_sha256
    └── created_at
```

The original project files remain independently stored in:

```text
files
```

The snapshot is therefore a historical execution artifact rather than the canonical project state.

---

# 22. Snapshot Queue Lifecycle

After the transaction commits:

```text
PostgreSQL
    │
    │ execution_job created
    ▼
Outbox / Queue
    │
    ▼
Redis
    │
    ▼
Evaluator Worker
```

The worker retrieves the job using:

```text
job_id
```

and loads the immutable snapshot.

The worker must not reconstruct the source by querying the current `files` table.

---

# 23. Sandbox Materialization

The evaluator transforms the persisted snapshot into an ephemeral sandbox workspace.

Example:

```text
snapshot
   │
   ▼
temporary workspace
   │
   ├── src/
   │   ├── main.cpp
   │   └── utils.cpp
   │
   └── build/
```

The workspace is disposable.

The sandbox must not write changes back into the project database.

---

# 24. Snapshot → Sandbox Boundary

The security boundary is:

```text
PostgreSQL snapshot
        │
        │ controlled service transfer
        ▼
Python Evaluator
        │
        │ validated execution payload
        ▼
C++ Sandbox Runtime
        │
        ▼
Ephemeral Container
```

The sandbox must not have direct access to:

```text
PostgreSQL
Redis
Host filesystem
Project storage
Other user projects
Docker socket
```

---

# 25. Snapshot During Compilation

The compiler operates exclusively on materialized snapshot contents.

Therefore:

```text
Compiler input
    =
Snapshot
```

not:

```text
Compiler input
    =
Current project files
```

This guarantees deterministic execution input.

---

# 26. Snapshot During Execution

The executable produced from the snapshot is executed under the resource policy attached to the job.

Example:

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

The resource policy must also be treated as immutable for the execution.

---

# 27. Snapshot Completion

When execution finishes:

```text
Sandbox
   │
   ├── stdout
   ├── stderr
   ├── exit code
   ├── signal
   ├── timing
   ├── memory
   └── resource violation
          │
          ▼
Execution Result
```

The snapshot itself is not modified.

The execution result references:

```text
execution_job.id
```

which indirectly identifies the snapshot used.

---

# 28. Snapshot Failure Handling

If compilation fails:

```text
SNAPSHOT
   ↓
COMPILING
   ↓
FAILED
```

The snapshot remains associated with the job.

If execution times out:

```text
SNAPSHOT
   ↓
RUNNING
   ↓
TIMEOUT
```

The snapshot remains immutable.

If a resource limit is exceeded:

```text
SNAPSHOT
   ↓
RUNNING
   ↓
RESOURCE_LIMIT
```

Again, the source snapshot remains unchanged.

---

# 29. Snapshot Cancellation

Cancellation does not modify the snapshot.

Example:

```text
QUEUED
  │
  └── CANCELLED
```

The immutable snapshot remains available according to retention policy.

For a job already running:

```text
RUNNING
   │
   ├── cancellation signal
   ▼
Sandbox terminated
   │
   ▼
CANCELLED
```

---

# 30. Snapshot Retention

Snapshots should have a configurable retention period.

Example initial policy:

```text
Active execution:
    retain indefinitely until completion

Completed execution:
    retain for 30 days

Failed execution:
    retain for 30 days

Cancelled execution:
    retain for 7 days

Expired execution:
    eligible for deletion
```

These are configurable policy values rather than hard-coded requirements.

---

# 31. Snapshot Garbage Collection

A background retention worker periodically identifies expired execution jobs.

Example:

```text
Retention Worker
      │
      ▼
Find expired jobs
      │
      ▼
Verify terminal state
      │
      ▼
Delete execution result
      │
      ▼
Delete execution job
      │
      ▼
Snapshot removed
```

Only terminal execution jobs may be garbage-collected.

Never delete snapshots belonging to:

```text
QUEUED
STARTING
COMPILING
RUNNING
```

---

# 32. Snapshot Deletion Safety

Deletion must be idempotent.

If the cleanup worker crashes:

```text
delete attempt
     ↓
worker crashes
     ↓
job remains
     ↓
next cleanup cycle
     ↓
delete again
```

The system must eventually converge to the desired retention state.

---

# 33. Snapshot Auditability

Every execution should be traceable through:

```text
request_id
    ↓
execution_job.id
    ↓
source_snapshot.snapshot_id
    ↓
snapshot_sha256
    ↓
execution_result.id
```

This allows operators to answer:

> Exactly what source code produced this execution result?

---

# 34. Snapshot Reproducibility

A historical execution should be reproducible using:

```text
snapshot
+
language
+
language_version
+
resource_policy
+
sandbox_version
```

The long-term production architecture should therefore additionally record:

```text
compiler_version
runtime_version
sandbox_image_digest
evaluator_version
```

Example:

```json
{
  "language": "cpp",
  "language_version": "17",
  "compiler_version": "gcc-14",
  "sandbox_image_digest": "sha256:...",
  "evaluator_version": "1.4.0"
}
```

This allows future investigation of differences caused by toolchain changes.

---

# 35. Execution Snapshot State Model

The logical snapshot state can be represented as:

```text
CAPTURED
   │
   ▼
VALIDATED
   │
   ▼
PERSISTED
   │
   ▼
QUEUED
   │
   ▼
DISPATCHED
   │
   ▼
MATERIALIZED
   │
   ▼
CONSUMED
   │
   ▼
RETAINED
   │
   ▼
EXPIRED
   │
   ▼
PURGED
```

The snapshot itself does not need a separate database status column for every state.

The authoritative lifecycle can be derived from:

```text
execution_jobs.status
+
timestamps
+
retention metadata
```

A dedicated snapshot status table may be introduced later if execution artifacts become independently managed.

---

# 36. Database Constraint Summary

The minimum production-oriented constraint set is:

```text
users
 ├── PK(id)
 ├── UNIQUE(email)
 ├── UNIQUE(username)
 └── CHECK(status)

projects
 ├── PK(id)
 ├── FK(owner_id → users.id)
 └── CHECK(visibility)

project_members
 ├── PK(project_id, user_id)
 ├── FK(project_id → projects.id)
 ├── FK(user_id → users.id)
 └── CHECK(role)

files
 ├── PK(id)
 ├── FK(project_id → projects.id)
 ├── FK(parent_id → files.id)
 ├── UNIQUE(project_id, path)
 └── CHECK(type / size)

execution_jobs
 ├── PK(id)
 ├── FK(project_id → projects.id)
 ├── FK(user_id → users.id)
 ├── FK(entry_file_id → files.id)
 ├── FK(entry_file_id, project_id → files.id, project_id)
 ├── CHECK(status)
 └── CHECK(timestamps)

execution_results
 ├── PK(id)
 ├── UNIQUE(job_id)
 ├── FK(job_id → execution_jobs.id)
 └── CHECK(non-negative metrics)

sessions
 ├── PK(id)
 ├── UNIQUE(token_hash)
 ├── FK(user_id → users.id)
 └── CHECK(expiry)

audit_events
 ├── PK(id)
 ├── FK(user_id → users.id)
 ├── FK(project_id → projects.id)
 └── CHECK(metadata)
```

---

# 37. Final Data Integrity Rule

Application code must never be considered the only source of integrity.

The architecture is:

```text
React
  │
  ▼
Node Gateway
  │
  ├── Authentication
  ├── Authorization
  └── Validation
        │
        ▼
PostgreSQL
  │
  ├── Primary Keys
  ├── Foreign Keys
  ├── Unique Constraints
  ├── Check Constraints
  └── Transaction Isolation
        │
        ▼
Evaluator
        │
        ▼
Sandbox
```

Each layer provides a different protection boundary.

The database protects **data integrity**.

The evaluator protects **execution correctness**.

The sandbox protects **host isolation**.

The gateway protects **API and authorization boundaries**.

---

# 38. Updated Data Architecture Principle

The most important execution rule is:

> **An execution runs against an immutable snapshot, never against mutable live project state.**

This guarantees:

* reproducible executions
* deterministic source selection
* historical debugging
* race-condition resistance
* safe asynchronous execution
* auditability
* separation between editing and execution
* compatibility with distributed workers
* future execution replay

---

# 39. Next Document

The next document in the CodeForge Cloud documentation set is:

```text
docs/04-api-reference.md
```

It should expose the database and execution model through versioned REST, WebSocket, and gRPC contracts.

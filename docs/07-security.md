# CodeForge Cloud

## Security Architecture & Threat Model

**Document:** 07 — Security
**Project:** CodeForge Cloud
**Subtitle:** Collaborative Real-Time Online Code Compiler & IDE Sandbox
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

---

# 1. Document Overview

CodeForge Cloud executes potentially untrusted source code submitted by users.

This makes security a fundamental architectural requirement rather than a secondary feature.

The platform must protect:

* the host operating system.
* sandbox infrastructure.
* other users.
* projects.
* source code.
* databases.
* Redis.
* internal services.
* authentication credentials.
* execution infrastructure.
* network resources.
* platform availability.

The primary security principle is:

> **Treat every user-submitted program as hostile.**

CodeForge Cloud therefore uses defense-in-depth security across the browser, API gateway, evaluator, execution scheduler, container runtime, operating system, database, network, and observability layers.

---

# 2. Security Objectives

The security architecture has six primary objectives:

1. Prevent arbitrary code from escaping its sandbox.
2. Prevent unauthorized access to projects and files.
3. Prevent denial-of-service through resource exhaustion.
4. Protect authentication and session credentials.
5. Protect internal services and infrastructure.
6. Maintain traceability of security-sensitive operations.

---

# 3. Security Principles

CodeForge Cloud follows these principles:

```text
Zero Trust
Least Privilege
Defense in Depth
Fail Closed
Secure by Default
Explicit Authorization
Isolation by Default
Immutable Execution
Resource Boundedness
Input Validation
Observable Security
```

---

# 4. Security Model

The platform contains multiple trust zones.

```text
┌──────────────────────────────────────────┐
│             UNTRUSTED ZONE               │
│                                          │
│ Browser / User Input / User Source Code  │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│          PUBLIC APPLICATION ZONE          │
│                                          │
│ Nginx → Node.js Gateway                  │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│           INTERNAL SERVICE ZONE          │
│                                          │
│ Evaluator / Redis / PostgreSQL / gRPC    │
└────────────────────┬─────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────┐
│        UNTRUSTED EXECUTION ZONE          │
│                                          │
│ C++ Sandbox / User Program               │
└──────────────────────────────────────────┘
```

Each boundary must enforce its own security controls.

---

# 5. Threat Model

The threat model assumes an attacker may be:

* an unauthenticated Internet user.
* an authenticated malicious user.
* a compromised user account.
* a malicious project member.
* a user submitting intentionally hostile code.
* a user attempting resource exhaustion.
* a user attempting cross-project access.
* a user attempting container escape.
* a user attempting to exploit a service vulnerability.

The architecture does not assume that user code is trustworthy.

---

# 6. Security Assets

Important assets include:

| Asset                  | Importance  |
| ---------------------- | ----------- |
| User credentials       | Critical    |
| Session tokens         | Critical    |
| Project source code    | High        |
| Execution snapshots    | High        |
| Database               | Critical    |
| Redis                  | High        |
| Sandbox host           | Critical    |
| Container runtime      | Critical    |
| Internal gRPC services | High        |
| API credentials        | Critical    |
| TLS private keys       | Critical    |
| Audit records          | High        |
| Execution results      | Medium/High |

---

# 7. Security Boundaries

The major security boundaries are:

```text
Internet
   │
   ▼
Nginx
   │
   ▼
Gateway
   │
   ├──────── PostgreSQL
   ├──────── Redis
   │
   ▼
Evaluator
   │
   ▼
Sandbox
   │
   ▼
User Program
```

Every transition must validate the data crossing the boundary.

---

# 8. Browser Security

The browser is fully untrusted.

The frontend must not be trusted for:

```text
authentication decisions
authorization decisions
resource limits
project ownership
execution permissions
file ownership
security policy
```

The browser can be modified by the user.

Therefore:

```text
UI restriction ≠ Security control
```

Example:

Hiding a Delete button is not authorization.

The server must still perform:

```text
authenticate
      ↓
authorize
      ↓
delete
```

---

# 9. Authentication

Authentication verifies the identity of the user.

The platform may use:

```text
Access token
Session token
Refresh token
```

depending on the final authentication implementation.

Passwords must never be stored directly.

Only password hashes should be persisted.

Recommended password hashing algorithms include modern password-hashing schemes such as:

```text
Argon2id
bcrypt
scrypt
```

The selected implementation must use an appropriate work factor.

---

# 10. Password Security

Passwords must:

* never be logged.
* never be returned through APIs.
* never be stored in plaintext.
* never be included in error messages.
* never be embedded in source code.

Registration flow:

```text
Password
   ↓
Validation
   ↓
Password Hash
   ↓
PostgreSQL
```

Login flow:

```text
Password
   ↓
Hash Verification
   ↓
Authentication Success
   ↓
Session / Access Token
```

---

# 11. Session Security

Session tokens must be treated as secrets.

The database stores:

```text
token_hash
```

rather than raw session tokens.

Sessions should contain:

```text
user_id
token_hash
created_at
expires_at
revoked_at
```

Expired or revoked sessions must not authenticate requests.

---

# 12. Token Security

Tokens should:

* have limited lifetime.
* be generated using a cryptographically secure random source.
* never be logged.
* never be placed in URLs.
* be transmitted only over HTTPS.
* be revoked when required.

For browser applications, token storage strategy must be selected carefully to reduce XSS and token-theft risk.

---

# 13. Authorization

Authentication answers:

> Who are you?

Authorization answers:

> What are you allowed to do?

Every protected resource must perform authorization independently.

Example:

```text
Request
   ↓
Authenticated User
   ↓
Project Membership
   ↓
Role
   ↓
Operation
```

---

# 14. Project Roles

The initial project roles are:

```text
OWNER
EDITOR
VIEWER
```

Recommended permission model:

| Operation      | OWNER | EDITOR | VIEWER |
| -------------- | ----: | -----: | -----: |
| View project   |     ✓ |      ✓ |      ✓ |
| Read files     |     ✓ |      ✓ |      ✓ |
| Create files   |     ✓ |      ✓ |      ✗ |
| Update files   |     ✓ |      ✓ |      ✗ |
| Delete files   |     ✓ |      ✓ |      ✗ |
| Execute code   |     ✓ |      ✓ | Policy |
| Manage members |     ✓ |      ✗ |      ✗ |
| Delete project |     ✓ |      ✗ |      ✗ |

Execution permission for viewers should be an explicit product policy rather than assumed.

---

# 15. Object-Level Authorization

Every resource request must verify ownership or membership.

Example:

```http
GET /api/v1/projects/{project_id}/files/{file_id}
```

The gateway must verify:

```text
Authenticated user
       ↓
Project exists
       ↓
User belongs to project
       ↓
File belongs to same project
       ↓
Permission granted
```

Never authorize solely from the supplied `file_id`.

---

# 16. Cross-Project Isolation

A critical security requirement is preventing cross-project access.

Example attack:

```text
User belongs to Project A

Request:
GET /projects/A/files/file-from-project-B
```

The request must fail.

Database relationships should reinforce this protection.

For example, composite constraints can ensure:

```text
file_id + project_id
```

refer to the same project.

---

# 17. ID Enumeration Protection

UUIDs are used for major resources.

Example:

```text
user_id
project_id
file_id
execution_id
job_id
snapshot_id
```

UUIDs reduce predictable identifier enumeration but do not replace authorization.

The platform must still verify authorization for every object.

---

# 18. Input Validation

All external input is untrusted.

Validate:

```text
path parameters
query parameters
headers
JSON body
file names
file paths
source code
resource policies
pagination
sorting
WebSocket events
```

Validation should occur before processing.

---

# 19. Request Size Limits

The gateway must enforce request limits.

Examples:

```text
Maximum JSON body
Maximum source file
Maximum project size
Maximum WebSocket message
Maximum execution request
Maximum batch size
```

Initial file limit:

```text
1 MB per file
```

Initial project limit:

```text
10 MB per project
```

These values should remain configurable.

---

# 20. Path Traversal Protection

File paths are security-sensitive.

Reject:

```text
../
../../
/etc/passwd
/home/user/secret
```

and equivalent encoded or normalized traversal attempts.

The system should normalize paths before validation.

A safe project path should remain inside the project workspace:

```text
/project/
    ├── src/
    ├── include/
    └── main.cpp
```

---

# 21. Filesystem Isolation

User code must not access the host filesystem.

Incorrect:

```text
Host filesystem
      │
      └── mounted into user container
```

Correct:

```text
Ephemeral workspace
      │
      ▼
User container
```

The workspace must be created specifically for the execution.

---

# 22. Docker Security

Containers must not run with unnecessary privileges.

Avoid:

```text
--privileged
```

Avoid mounting:

```text
/var/run/docker.sock
```

into user workloads.

Avoid host filesystem mounts.

User containers should receive only the minimum resources and capabilities required.

---

# 23. Linux Namespace Isolation

The sandbox should use appropriate Linux namespaces.

Relevant namespaces include:

```text
PID
Mount
Network
UTS
IPC
User
```

Namespaces reduce visibility into host resources.

The exact namespace configuration must be tested on the deployment environment.

---

# 24. Process Isolation

User programs should run in isolated process namespaces.

The user must not be able to inspect unrelated host processes.

Example:

```text
Host
 ├── Gateway
 ├── PostgreSQL
 ├── Evaluator
 └── User Sandbox
       └── User Process
```

The user process should see only the resources intentionally exposed to it.

---

# 25. Resource Limits

Resource exhaustion is one of the largest threats to an online compiler.

The sandbox must limit:

```text
CPU
Memory
Wall-clock time
Process count
Disk usage
Output size
Source size
```

Example policy:

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

These are initial example limits and should be centrally controlled.

---

# 26. CPU Protection

CPU limits prevent programs such as:

```cpp
while (true)
{
}
```

from consuming a worker indefinitely.

CPU controls should include:

```text
CPU quota
CPU time
wall-clock timeout
```

Both CPU time and wall-clock time are useful because they address different failure modes.

---

# 27. Memory Protection

Memory limits prevent programs from consuming excessive RAM.

Example:

```cpp
while (true)
{
    new char[1024 * 1024];
}
```

The sandbox must terminate workloads that exceed the configured memory limit.

---

# 28. Process Limits

Programs may attempt to create large numbers of processes.

The sandbox must enforce a process limit.

Conceptually:

```text
Maximum processes = N
```

When the limit is exceeded:

```text
RESOURCE_LIMIT
```

should be returned.

---

# 29. Disk Limits

User programs must not fill the host disk.

Temporary execution storage must have:

```text
maximum size
```

and cleanup must occur after execution.

The system should reject or terminate workloads that exceed disk limits.

---

# 30. Output Limits

A malicious program can generate unlimited output.

Example:

```cpp
while (true)
{
    std::cout << "AAAA";
}
```

The sandbox must enforce an output limit.

Possible behavior:

```text
Output exceeds limit
       ↓
Stop capture
       ↓
Terminate process
       ↓
RESOURCE_LIMIT
```

or a controlled output-truncation result, depending on product policy.

---

# 31. Timeout Protection

Every execution must have a deadline.

Example:

```text
Execution
   ↓
5-second wall timeout
   ↓
Process terminated
   ↓
TIMEOUT
```

Timeouts must be enforced independently of user code.

The user cannot disable them.

---

# 32. Network Isolation

Network access should be disabled for user execution by default.

The initial sandbox policy is:

```text
User Program
     │
     X
 Internet
```

This prevents:

* external network scanning.
* data exfiltration.
* attacks against internal services.
* cryptocurrency mining pools.
* abuse of third-party services.

Future network-enabled execution, if introduced, requires a separate security design.

---

# 33. Internal Network Protection

User containers must not reach:

```text
PostgreSQL
Redis
Gateway internal ports
Evaluator
Docker daemon
Host services
Cloud metadata services
```

Network isolation must be tested explicitly.

---

# 34. Cloud Metadata Protection

If CodeForge Cloud is deployed on a cloud platform, user workloads must not access instance or container metadata endpoints.

The platform should use:

* network isolation.
* metadata endpoint protection.
* workload identity controls.
* restricted routing.

This is especially important in cloud deployments.

---

# 35. Capability Restrictions

Linux capabilities should be minimized.

User workloads should not receive unnecessary capabilities such as:

```text
CAP_SYS_ADMIN
CAP_SYS_PTRACE
CAP_NET_ADMIN
CAP_SYS_MODULE
```

The final sandbox profile should explicitly define allowed capabilities rather than relying on broad defaults.

---

# 36. Seccomp

seccomp should restrict dangerous system calls.

The sandbox should use a deny-by-default or carefully reviewed syscall profile where practical.

The profile must support:

```text
compiler
runtime
standard libraries
normal C/C++ execution
```

while restricting unnecessary kernel functionality.

---

# 37. Read-Only Filesystem

Where possible, the container root filesystem should be read-only.

Example:

```text
Container Root
    READ ONLY

Temporary Workspace
    READ/WRITE
```

This reduces persistence and tampering opportunities.

---

# 38. Temporary Workspace

Every execution receives a unique workspace.

Example:

```text
/workspaces/
    execution-UUID/
```

Lifecycle:

```text
Create
  ↓
Materialize snapshot
  ↓
Compile
  ↓
Execute
  ↓
Capture result
  ↓
Delete
```

Workspace cleanup must occur even when execution fails.

---

# 39. Snapshot Security

Execution snapshots contain user source code.

They must be protected against:

* unauthorized access.
* modification.
* cross-project access.
* accidental logging.
* unintended retention.

Each execution must reference its own immutable snapshot.

---

# 40. Snapshot Integrity

Each source file should have a SHA-256 hash.

The snapshot should also have an aggregate hash.

Example:

```text
main.cpp
   ↓
SHA-256

utils.cpp
   ↓
SHA-256

Sorted file hashes
   ↓
Snapshot SHA-256
```

The hash helps verify that the snapshot executed is the snapshot that was requested.

---

# 41. Immutable Execution

Once execution begins, the following must not change:

```text
project_id
user_id
entry_file
language
source_snapshot
resource_policy
submitted_at
```

Only lifecycle fields may change:

```text
status
started_at
completed_at
```

This prevents execution tampering.

---

# 42. Compiler Security

The compiler itself is part of the attack surface.

A malicious program may attempt to exploit:

```text
compiler bugs
preprocessor behavior
template expansion
filesystem access
resource exhaustion
compiler subprocesses
```

Compilation must therefore occur inside the same security boundary or an appropriately isolated compilation environment.

---

# 43. Compiler Flags

The sandbox should use controlled compiler configuration.

Users should not be able to arbitrarily modify security-critical compiler options.

Compiler execution should be bounded by:

```text
CPU
memory
wall-clock time
process count
filesystem
output
```

---

# 44. Compiler Process Tree

Compilation may create subprocesses.

Therefore resource limits must apply to the entire process tree.

Incorrect:

```text
Compiler
  └── Child process
       └── Escape resource limit
```

Correct:

```text
Execution cgroup
      │
      ├── Compiler
      ├── Compiler children
      └── Runtime
```

---

# 45. Command Injection Protection

Never construct shell commands directly from user-controlled strings.

Unsafe pattern:

```text
shell("g++ " + user_input)
```

Prefer structured process execution with explicitly controlled arguments.

User input must never determine arbitrary executable paths.

---

# 46. Shell Restrictions

A user should not receive unrestricted host shell access.

The terminal in the Web IDE is a controlled application feature, not a host terminal.

Any shell-like functionality must execute inside an isolated workspace with the same or stronger restrictions as code execution.

---

# 47. SQL Injection Protection

Database queries must use parameterized queries.

Unsafe:

```text
SELECT * FROM users WHERE username = '<user_input>';
```

Safe:

```text
Prepared statement
        +
Bound parameter
```

ORM/query-builder parameterization should be used where available.

---

# 48. XSS Protection

User-controlled content may appear in:

```text
file names
project names
display names
terminal output
compiler errors
execution results
comments
collaboration messages
```

The frontend must treat these as untrusted data.

Never render user-controlled HTML without explicit sanitization.

---

# 49. Terminal Output Security

Compiler and program output may contain terminal escape sequences.

The terminal component should safely handle:

```text
ANSI sequences
control characters
very long lines
binary-like output
malformed UTF-8
```

It should not allow output to execute browser-side JavaScript.

---

# 50. WebSocket Security

WebSocket connections must be authenticated.

Connection flow:

```text
WebSocket Request
      ↓
Authentication
      ↓
Authorization
      ↓
Project Room
```

Every project room must be access-controlled.

---

# 51. WebSocket Message Validation

Every event should validate:

```text
event type
event ID
project ID
file ID
user ID where applicable
payload size
payload structure
revision
```

Malformed events should be rejected.

---

# 52. Collaboration Security

A malicious collaborator must not be able to modify files outside their authorized project.

Example:

```text
Project A room
      ↓
User A
      ↓
file:update(Project B)
```

The gateway must reject the event.

Client-provided `user_id` should never be trusted.

The server should derive identity from the authenticated session.

---

# 53. Collaboration Rate Limiting

WebSocket clients should be rate limited.

Protect against:

```text
event flooding
cursor flooding
file-update flooding
presence flooding
large messages
connection flooding
```

Use Redis-backed rate limiting where appropriate.

---

# 54. API Rate Limiting

Sensitive endpoints should have rate limits.

Especially:

```text
login
registration
execution submission
file creation
project creation
WebSocket connection
```

Execution submission deserves particularly strong protection because execution consumes compute resources.

---

# 55. Denial-of-Service Protection

DoS protection must exist at multiple layers.

```text
Nginx
   ↓
Gateway rate limiting
   ↓
Evaluator queue limits
   ↓
Sandbox resource limits
```

No single layer should be the only defense.

---

# 56. Execution Queue Protection

The evaluator should enforce:

```text
maximum queue depth
per-user execution rate
per-project execution rate
worker concurrency
global concurrency
```

If capacity is exhausted:

```text
RESOURCE_EXHAUSTED
```

or the equivalent API response should be returned.

---

# 57. Fair Scheduling

A single user must not monopolize all workers.

Possible scheduling policy:

```text
Global concurrency
       +
Per-user quota
       +
Per-project quota
       +
Queue fairness
```

Future versions may implement weighted or priority scheduling.

---

# 58. Authentication Abuse Protection

Login endpoints should be protected against brute-force attacks.

Controls may include:

```text
rate limiting
progressive delays
temporary lockouts
IP-based controls
account-based controls
monitoring
```

Care must be taken to avoid making account enumeration easier.

---

# 59. Account Enumeration Protection

Authentication responses should avoid revealing unnecessary information.

For example, avoid clearly distinguishing:

```text
User does not exist
```

from:

```text
Password incorrect
```

when such distinction would enable account enumeration.

---

# 60. CSRF Protection

If browser authentication uses cookies, CSRF protections must be implemented.

Possible mechanisms include:

```text
SameSite cookies
CSRF tokens
Origin validation
```

If bearer tokens are used without cookies, the CSRF threat model differs, but XSS and token theft remain important concerns.

---

# 61. CORS

CORS should allow only trusted frontend origins.

Avoid:

```text
Access-Control-Allow-Origin: *
```

for authenticated production APIs unless explicitly justified.

Allowed origins should be configuration-driven.

---

# 62. Security Headers

Nginx or the application should configure appropriate security headers.

Examples include:

```text
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
Strict-Transport-Security
```

The exact policy must be tested against the frontend.

---

# 63. HTTPS

Production traffic should use HTTPS.

Expected flow:

```text
Browser
   │ HTTPS
   ▼
Nginx
   │
   ▼
Gateway
```

HTTP should redirect to HTTPS where appropriate.

---

# 64. TLS for Internal Services

Production internal gRPC communication should use authenticated encryption where required.

Recommended future architecture:

```text
Gateway
   │ mTLS
   ▼
Evaluator
   │ mTLS
   ▼
Sandbox
```

Service identity should be independently verified.

---

# 65. Database Security

PostgreSQL should:

* use strong authentication.
* use encrypted connections where required.
* restrict network access.
* use least-privilege database accounts.
* separate development and production databases.
* prevent public Internet exposure.

The sandbox must never have direct database credentials.

---

# 66. Database Least Privilege

Separate service identities where practical.

Example:

```text
Gateway DB User
Evaluator DB User
Migration DB User
```

The migration identity may have elevated schema privileges.

Application identities should have only required privileges.

---

# 67. Redis Security

Redis must not be publicly exposed.

Restrict Redis access to trusted services.

Use:

```text
network isolation
authentication
TLS where appropriate
ACLs
```

depending on deployment.

User workloads must never receive Redis credentials.

---

# 68. Docker Daemon Security

The Docker daemon is highly privileged.

Therefore:

```text
User Code
   X
Docker Socket
```

The Docker socket must never be mounted into user containers.

Only trusted infrastructure components may interact with the container runtime.

---

# 69. Host Security

The sandbox host should be hardened.

Recommended controls include:

```text
minimal OS packages
automatic security updates
restricted SSH
firewall
non-root services where practical
disk monitoring
resource monitoring
centralized logs
```

---

# 70. Container Image Security

Execution images should be:

* minimal.
* version-pinned.
* regularly rebuilt.
* vulnerability scanned.
* free of unnecessary packages.

Avoid installing:

```text
curl
wget
ssh
network tools
debuggers
compilers unrelated to the selected language
```

unless required.

---

# 71. Image Immutability

Production workloads should reference immutable image versions or digests.

Example:

```text
sandbox@sha256:<digest>
```

rather than relying only on mutable tags.

The image digest should be recorded with executions when reproducibility is required.

---

# 72. Supply Chain Security

Dependencies and container images are part of the attack surface.

Security practices should include:

```text
dependency scanning
container scanning
lock files
trusted registries
version pinning
signature verification where available
```

---

# 73. Dependency Management

Dependencies should be reviewed before introduction.

For each dependency consider:

```text
maintainer activity
license
known vulnerabilities
transitive dependencies
permissions
maintenance cost
```

Unused dependencies should be removed.

---

# 74. Secrets Management

Secrets include:

```text
database passwords
JWT secrets
session secrets
TLS private keys
API keys
service credentials
registry credentials
```

Secrets must be externalized.

Never store production secrets in:

```text
Git
Dockerfile
README
source code
logs
```

---

# 75. Secret Rotation

Secrets should be rotatable.

Examples:

```text
database credentials
API keys
JWT signing keys
service certificates
```

Rotation procedures should avoid unnecessary downtime.

---

# 76. Logging Security

Security-sensitive logs should include:

```text
timestamp
request_id
user_id where appropriate
project_id where appropriate
execution_id
event
result
```

Do not log:

```text
password
access token
session token
private key
database password
raw source code by default
```

---

# 77. Audit Logging

Important security events should generate audit records.

Examples:

```text
login
logout
failed authentication
project creation
project deletion
membership changes
permission changes
execution submission
execution cancellation
security violation
```

Audit records should be protected against unauthorized modification.

---

# 78. Audit Event Example

Conceptual event:

```json
{
  "event_type": "EXECUTION_SUBMITTED",
  "user_id": "uuid",
  "project_id": "uuid",
  "metadata": {
    "execution_id": "uuid",
    "snapshot_id": "uuid"
  }
}
```

Do not store raw credentials or unnecessary source content in audit metadata.

---

# 79. Security Monitoring

Monitor for:

```text
authentication failures
execution abuse
resource-limit violations
queue abuse
authorization failures
unexpected container behavior
network violations
repeated sandbox failures
```

---

# 80. Security Metrics

Useful metrics include:

```text
auth_failure_total
authorization_denied_total
rate_limit_total
execution_timeout_total
execution_resource_limit_total
sandbox_failure_total
container_start_failure_total
security_event_total
```

---

# 81. Alerting

Alerts should be created for abnormal behavior.

Examples:

```text
Sudden authentication failures
Large increase in execution failures
Repeated resource exhaustion
Unexpected network attempts
Sandbox crash spike
Database connection anomaly
Redis outage
```

---

# 82. Incident Response

When a serious security event occurs:

```text
Detect
  ↓
Contain
  ↓
Investigate
  ↓
Eradicate
  ↓
Recover
  ↓
Review
```

The exact procedure should be documented operationally before production launch.

---

# 83. Sandbox Incident Response

If sandbox escape is suspected:

1. Stop affected workers.
2. Prevent new executions on affected hosts.
3. Preserve relevant logs.
4. Isolate the host.
5. Rotate compromised credentials if necessary.
6. Investigate the container/image.
7. Rebuild from trusted artifacts.
8. Patch the vulnerability.
9. Re-run security tests.
10. Restore service only after verification.

---

# 84. Compromised Container Response

A compromised execution container should be treated as disposable.

```text
Compromised container
       ↓
Terminate
       ↓
Delete workspace
       ↓
Destroy container
       ↓
Replace worker
```

Do not attempt to continue using a potentially compromised container.

---

# 85. Worker Isolation

Workers should ideally be disposable.

Preferred:

```text
Job
 ↓
Fresh sandbox
 ↓
Execution
 ↓
Result
 ↓
Destroy sandbox
```

Avoid long-lived containers containing multiple unrelated user workloads.

---

# 86. Data Retention

Security and privacy require controlled retention.

Execution artifacts should not remain indefinitely.

Example configurable policy:

```text
Completed execution: 30 days
Failed execution: 30 days
Cancelled execution: 7 days
Expired execution: purge eligible
```

Retention must comply with the platform's final legal and product requirements.

---

# 87. Secure Deletion

When execution artifacts expire:

```text
Snapshot
   ↓
Retention check
   ↓
Delete
```

Temporary sandbox workspaces should be destroyed immediately after execution where practical.

---

# 88. Backup Security

Backups contain sensitive information.

Backups must:

* be encrypted.
* have restricted access.
* have retention limits.
* be tested for restoration.
* not be publicly accessible.

---

# 89. Multi-Tenant Security

Projects are logical tenants.

Every project-scoped operation must enforce:

```text
user → project membership → resource
```

No tenant should access another tenant's:

```text
files
snapshots
executions
results
collaboration rooms
audit information
```

---

# 90. Tenant Isolation in Redis

Redis keys should include project or resource identifiers.

Example:

```text
codeforge:project:<project_id>:presence
codeforge:project:<project_id>:events
codeforge:execution:<execution_id>:status
```

Access should still be controlled at the application layer.

---

# 91. Tenant Isolation in Storage

If object storage is introduced, use project-scoped namespaces.

Example:

```text
projects/<project_id>/snapshots/<snapshot_id>
```

Never allow clients to directly construct arbitrary object-storage paths.

---

# 92. Secure API Error Handling

Errors must reveal enough information for clients to recover without exposing internals.

Good:

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "The requested project does not exist.",
    "request_id": "uuid"
  }
}
```

Bad:

```text
PostgreSQL exception:
connection failed to 10.x.x.x
password authentication failed
```

---

# 93. Security Response Codes

Common responses:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
413 Payload Too Large
422 Unprocessable Entity
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# 94. Security Testing Program

Security testing should cover:

```text
Authentication
Authorization
Input validation
API security
WebSocket security
Sandbox isolation
Filesystem isolation
Network isolation
Resource limits
Container security
Dependency security
Secret handling
Logging
Data isolation
```

---

# 95. Sandbox Escape Testing

The security suite should attempt to verify isolation against:

```text
/proc access
host filesystem access
Docker socket access
network access
process inspection
privilege escalation
capability abuse
syscall abuse
namespace escape
resource exhaustion
```

These tests should be executed in a controlled environment.

---

# 96. Example Security Test Cases

### Cross-project access

```text
User A
  ↓
Project A
  ↓
Request Project B file
  ↓
403 / 404
```

### Path traversal

```text
../../etc/passwd
       ↓
Rejected
```

### Infinite loop

```cpp
while (true) {}
```

Expected:

```text
TIMEOUT
```

### Memory exhaustion

```text
Large allocation
       ↓
Memory limit
       ↓
RESOURCE_LIMIT
```

### Output exhaustion

```text
Infinite output
       ↓
Output limit
       ↓
RESOURCE_LIMIT
```

---

# 97. Security Regression Tests

Every discovered security vulnerability should produce a permanent regression test.

Workflow:

```text
Vulnerability
      ↓
Fix
      ↓
Regression Test
      ↓
CI
```

This prevents the same vulnerability from returning.

---

# 98. CI Security Gates

CI should eventually include:

```text
Unit tests
Lint
Static analysis
Dependency scanning
Secret scanning
Container scanning
Integration tests
Security tests
```

A critical security failure should block release.

---

# 99. Secure Development Lifecycle

The development lifecycle should be:

```text
Requirement
    ↓
Threat Modeling
    ↓
Architecture
    ↓
Implementation
    ↓
Security Review
    ↓
Testing
    ↓
Security Testing
    ↓
Code Review
    ↓
Release
```

Security must not be added only after implementation.

---

# 100. Security Review Checklist

Before merging security-sensitive code:

```text
[ ] Authentication reviewed
[ ] Authorization reviewed
[ ] Input validation reviewed
[ ] Resource limits reviewed
[ ] Error handling reviewed
[ ] Logging reviewed
[ ] Secrets reviewed
[ ] Dependency impact reviewed
[ ] Sandbox impact reviewed
[ ] Network impact reviewed
[ ] Database impact reviewed
[ ] Tests added
```

---

# 101. Threat-to-Control Matrix

| Threat                   | Primary Control            | Secondary Control          |
| ------------------------ | -------------------------- | -------------------------- |
| Arbitrary code execution | Sandbox                    | seccomp/namespaces         |
| Container escape         | Least privilege            | Kernel/container hardening |
| CPU exhaustion           | CPU limits                 | Queue limits               |
| Memory exhaustion        | cgroups                    | Scheduler limits           |
| Fork bomb                | Process limit              | Worker isolation           |
| Output flooding          | Output limit               | Gateway limits             |
| Disk exhaustion          | Disk quota                 | Cleanup                    |
| Network abuse            | Network disabled           | Firewall                   |
| Cross-project access     | Authorization              | DB constraints             |
| SQL injection            | Parameterized queries      | Validation                 |
| XSS                      | Output encoding/CSP        | Sanitization               |
| CSRF                     | SameSite/CSRF              | Origin validation          |
| Credential theft         | Secure sessions            | HTTPS                      |
| Brute force              | Rate limiting              | Monitoring                 |
| Queue abuse              | Fair scheduling            | Quotas                     |
| Secret exposure          | Secret management          | Log filtering              |
| Dependency attack        | Scanning                   | Pinning                    |
| WebSocket abuse          | Authentication/rate limits | Message validation         |

---

# 102. Security Architecture Summary

The security model can be represented as:

```text
                        Internet
                           │
                           ▼
                     ┌───────────┐
                     │   Nginx   │
                     └─────┬─────┘
                           │
                     TLS / Limits
                           │
                           ▼
                     ┌───────────┐
                     │  Gateway  │
                     └─────┬─────┘
                           │
                Auth / AuthZ / Validation
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         PostgreSQL      Redis       Evaluator
                                        │
                                  Policy Validation
                                        │
                                        ▼
                                  ┌───────────┐
                                  │  Sandbox  │
                                  └─────┬─────┘
                                        │
                              ┌─────────┴─────────┐
                              ▼                   ▼
                         Namespaces           cgroups
                              │                   │
                              └─────────┬─────────┘
                                        ▼
                                     seccomp
                                        │
                                        ▼
                                  User Program
```

---

# 103. Defense-in-Depth Model

No single security control should be considered sufficient.

The sandbox uses multiple layers:

```text
Application validation
        ↓
Authorization
        ↓
Execution policy
        ↓
Ephemeral workspace
        ↓
Container isolation
        ↓
Namespaces
        ↓
cgroups
        ↓
Capability restrictions
        ↓
seccomp
        ↓
Network isolation
        ↓
Read-only filesystem
        ↓
Process timeout
        ↓
Output limits
        ↓
Container destruction
```

Failure of one layer should not automatically result in host compromise.

---

# 104. Security Invariants

The following invariants must always hold.

### Invariant 1

User code never executes in the Gateway.

### Invariant 2

User code never executes directly on the host.

### Invariant 3

User code cannot access another project's data.

### Invariant 4

User code cannot access PostgreSQL.

### Invariant 5

User code cannot access Redis.

### Invariant 6

User code cannot access the Docker socket.

### Invariant 7

User code cannot access the host filesystem.

### Invariant 8

Every execution has resource limits.

### Invariant 9

Every execution has a timeout.

### Invariant 10

Every execution runs against an immutable snapshot.

### Invariant 11

Every protected operation performs authorization.

### Invariant 12

Security-sensitive failures fail closed.

---

# 105. Security Decision Rules

When choosing between two implementation approaches, prefer the one that:

```text
reduces privileges
reduces attack surface
reduces trust assumptions
reduces persistent state
reduces network access
reduces exposed interfaces
improves observability
improves isolation
```

---

# 106. Future Security Enhancements

Future production versions may introduce:

```text
mTLS between services
Kubernetes NetworkPolicies
Pod Security Standards
gVisor
Kata Containers
microVM isolation
SELinux/AppArmor
image signing
SBOM generation
runtime threat detection
centralized SIEM
WAF
advanced bot protection
automated vulnerability scanning
hardware-backed secrets
```

These are future enhancements and should not be considered implemented by the initial architecture.

---

# 107. Production Security Baseline

Before production deployment, the platform should satisfy at least:

```text
[ ] HTTPS enabled
[ ] Strong authentication
[ ] Authorization enforced
[ ] Rate limiting enabled
[ ] PostgreSQL private
[ ] Redis private
[ ] Sandbox network disabled
[ ] Docker socket inaccessible
[ ] Host filesystem inaccessible
[ ] Resource limits enabled
[ ] Timeout enabled
[ ] Process limits enabled
[ ] Output limits enabled
[ ] seccomp enabled
[ ] Namespace isolation enabled
[ ] Capability restrictions enabled
[ ] Secrets externalized
[ ] Security logging enabled
[ ] Audit logging enabled
[ ] Dependency scanning enabled
[ ] Container scanning enabled
[ ] Backup encryption enabled
[ ] Security regression tests passing
```

---

# 108. Final Security Architecture

The fundamental security boundary is:

```text
                   TRUSTED PLATFORM
                          │
         ┌────────────────┼────────────────┐
         │                │                │
      Gateway          Evaluator       Database
         │                │
         │                │
         └────────┬───────┘
                  │
            SECURITY BOUNDARY
                  │
                  ▼
          UNTRUSTED EXECUTION
                  │
             ┌────┴────┐
             │ Sandbox │
             └────┬────┘
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
   Namespace    cgroups    seccomp
       │          │          │
       └──────────┼──────────┘
                  ▼
            User Program
```

The platform's most important rule remains:

> **User-submitted code is hostile by default and must never be trusted with platform privileges.**

---

# 109. Document Status

**Document:** 07 — Security
**Version:** 1.0
**Status:** Draft
**Author:** Adarsh Kumar

**Previous Document:** `06-development-guide.md`

**Next Document:** `08-gap-analysis.md`

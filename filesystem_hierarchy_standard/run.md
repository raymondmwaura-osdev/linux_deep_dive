# The `/run` Directory in Linux: A Deep Architectural Analysis

## Introduction

The `/run` directory is the **authoritative location for volatile runtime state** in modern Linux systems. It exists to hold data that is:

* Required during system operation
* Generated dynamically at runtime
* Invalid across reboots
* Needed early in the boot process

From an operating system architecture perspective, `/run` functions as the system’s **ephemeral control plane**, mediating coordination between the kernel, init system, daemons, and user-space processes during live execution.

---

## Motivation for `/run`

### The Historical Problem

Historically, runtime state was stored under:

* `/var/run`
* `/var/lock`

This design was flawed because:

* `/var` may reside on disk and not be available during early boot
* Stale files could persist across crashes or reboots
* Cleanup ordering was unreliable
* Runtime correctness depended on filesystem hygiene

These weaknesses became unacceptable as systems grew more dynamic and parallelized.

---

### The Architectural Solution

`/run` was introduced to solve these issues by providing:

* A **guaranteed-early-mounted filesystem**
* A **clean namespace on every boot**
* A **single, unambiguous location for runtime state**

In most systems, `/var/run` and `/var/lock` are now symbolic links to `/run` and `/run/lock`, respectively.

---

## `/run` as a Volatile Filesystem

### tmpfs Backing

`/run` is almost universally mounted as **`tmpfs`**, an in-memory filesystem:

* Contents reside in RAM (with optional swap backing)
* Data is destroyed on reboot
* Read/write operations are fast and low-latency

This backing enforces the invariant that **runtime state must never outlive the runtime**.

---

### Boot-Time Availability

`/run` is mounted very early in the boot sequence (often before `/var` or `/usr`) making it suitable for:

* PID files
* UNIX domain sockets
* Early inter-process communication
* System coordination primitives

This early availability is one of its defining characteristics.

---

## Semantic Scope of `/run`

### What Belongs in `/run`

Data placed in `/run` must satisfy all of the following:

* Required only while the system is running
* Automatically regenerable
* Unsafe or meaningless if persisted
* Needed by multiple processes or subsystems

Canonical examples include:

* Process ID (PID) files
* Daemon control sockets
* Lock files
* State markers indicating service readiness

---

### What Does *Not* Belong in `/run`

Architecturally incorrect uses include:

* Configuration files
* Logs
* Caches
* User data
* Any state requiring persistence across reboots

Violating this boundary leads to nondeterministic behavior and boot-time ambiguity.

---

## Structural Organization of `/run`

Although FHS does not rigidly prescribe internal layout, conventions are well established.

### Common Subdirectories

| Path               | Function                      |
| ------------------ | ----------------------------- |
| `/run/<service>/`  | Service-specific runtime data |
| `/run/lock/`       | Advisory lock files           |
| `/run/user/<uid>/` | Per-user runtime state        |
| `/run/systemd/`    | Init system coordination data |

Each subdirectory corresponds to a **coordination domain**.

---

### Namespacing and Ownership

* Services are expected to create their own subdirectories
* Ownership and permissions reflect trust boundaries
* Flat layouts are discouraged due to collision risk

This structure supports parallel service startup and isolation.

---

## `/run/user`: Per-User Runtime State

### Purpose

`/run/user/<uid>` provides **session-scoped runtime storage** for user processes.

Typical contents include:

* IPC sockets
* Session buses
* Temporary authentication artifacts

These directories are usually created at login and destroyed at logout.

---

### Security and Isolation

* Owned by the user
* Permissioned to prevent cross-user access
* Isolated from system-level runtime state

This design supports multi-user systems without leaking runtime capabilities.

---

## `/run` and systemd

### PID Files and Service Coordination

In systemd-based systems:

* PID files are often unnecessary
* systemd tracks processes directly
* `/run` still hosts readiness markers, sockets, and state files

`/run` thus serves as a **shared memory substrate** for service orchestration.

---

### Socket Activation

Systemd socket activation places listening sockets in `/run`:

* Services can be started on demand
* Sockets persist independently of service lifetimes
* Restart semantics become trivial

This pattern depends critically on `/run`’s volatility and early availability.

---

## `/run` and Locking Semantics

### Advisory Locks

`/run/lock` contains lock files used to coordinate access to shared resources.

Properties:

* Locks disappear automatically on reboot
* No risk of stale locks
* Filesystem semantics enforce atomicity

This makes `/run/lock` superior to disk-backed locking locations.

---

## `/run` in Containers and Namespaces

### Containers

In containerized environments:

* `/run` is often container-private
* Used for intra-container coordination
* Not suitable for cross-container persistence

This reinforces `/run` as **instance-local state**, not system identity.

---

### Mount and PID Namespaces

`/run` participates in namespace isolation:

* Different views per container or process tree
* Prevents state leakage
* Enables fine-grained lifecycle management

---

## Failure Modes and Recovery Semantics

### Crash Safety

Because `/run` is volatile:

* System crashes automatically clear corrupted state
* Reboot restores invariants
* No manual cleanup is required

This is a deliberate trade-off favoring **deterministic recovery** over persistence.

---

### Memory Pressure

As a tmpfs:

* Excessive use of `/run` consumes RAM
* Misbehaving services can induce memory pressure
* Monitoring is essential in constrained environments

However, this pressure is explicit and observable, unlike silent disk growth.

---

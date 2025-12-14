# Linux Filesystem Hierarchy Standard (FHS)

## Introduction and Scope

The **Filesystem Hierarchy Standard (FHS)** defines the directory structure and directory content for UNIX-like operating systems, including Linux. The standard’s primary objective is to promote interoperability of applications, system administration tools, development tools, and documentation by specifying where files and directories should reside in a compliant system. The current authoritative specification is **FHS Version 3.0** (also published as a FreeDesktop.org standard in late 2025).

From a systems architecture perspective, FHS serves as a blueprint that:

* Encapsulates historical UNIX filesystem conventions.
* Separates data based on *static vs variable* and *essential vs non-essential* usage.
* Supports system booting, recovery, and operation uniformly across distributions.

Compliance with FHS is essential for software portability, predictable filesystem layout, and ease of system administration.

---

## Conceptual Framework of FHS

### Hierarchical Structure

Linux uses a **single, rooted directory hierarchy** where all files and directories descend from the **root directory (`/`)**. Unlike operating systems with multiple root partitions, Linux integrates all mount points within this unified namespace.

### Core Design Principles

FHS groups filesystem directories based on *purpose* and *stability*:

+ **Essential Binaries/Libraries**; Required for boot and early system operation.
+ **Static Data**; Files that do not change during normal operation.
+ **Variable Data**; Files that change frequently (logs, spools, caches).
+ **Secondary/Tertiary Hierarchies**; Shareable or host-specific data (e.g., `/usr`, `/usr/local`).

Standard design recognizes that:

* Some directories can be mounted read-only.
* Variable data must be isolated to prevent corruption and facilitate maintenance.

---

## The Root Filesystem (`/`)

The root directory `/` is the highest level of the filesystem hierarchy. It contains essential directories needed for system boot, repair, and minimal operation. Only FHS-specified directories (and their symbolic links) should exist at this level unless carefully justified for application portability concerns.

### Required Root Directories

+ `/bin`; Essential command binaries required in single-user or rescue mode.
+ `/boot`; Bootloader and kernel images (e.g., GRUB, initramfs).
+ `/dev`; Device files representing hardware and virtual devices.
+ `/etc`; Host-specific configuration files (non-binary).
+ `/lib`; Shared libraries required by `/bin` and `/sbin`.
+ `/media`; Mount points for removable media (e.g., USB, CD/DVD).
+ `/mnt`; Generic temporary mount point for administrators.
+ `/opt`; Third-party or optional package installations.
+ `/run`; Transient runtime data (e.g., PIDs, sockets).
+ `/sbin`; Essential system binaries used by administrators.
+ `/srv`; Data for services hosted on the system (e.g., HTTP/FTP roots).
+ `/tmp`; Temporary files (cleared periodically or on reboot).
+ `/usr`; Secondary hierarchy.
+ `/var`; Variable data.

Optional based on subsystem installation:

* `/home`; User home directories.
* `/root`; Home directory for the root account.
* `/lib<qualifier>`; Architecture-specific or alternate format libraries.

**Key Compliance Note:** Third-party software should not create new top-level directories outside of those defined in FHS without rigorous justification, as doing so undermines portability and system consistency.

---

## The `/usr` Hierarchy

### Purpose

`/usr` holds the **secondary system hierarchy**. It is intended for **shareable, static, read-only data**; including the bulk of user utilities and application binaries. It may be mounted read-only and shared across systems.

### Standard Subdirectories in `/usr`

+ `/usr/bin`; Most user command binaries (non-essential at boot).
+ `/usr/lib`; Libraries for `/usr/bin` and `/usr/sbin`.
+ `/usr/sbin`; Non-essential system binaries.
+ `/usr/share`; Architecture-independent data (docs, icons, locales).
+ `/usr/local`; Local administrator installations (host-specific).

Optional or specific:

* `/usr/include`; Header files for development.
* `/usr/libexec`; Program binaries invoked by other programs.
* `/usr/games`; Game binaries.

**Design Note:** Major software packages must not install directly under `/usr` root (e.g., not directly into `/usr/<app>`). Rather, they employ structured subdirectories.

---

## The `/var` Hierarchy

### Purpose

Unlike `/usr`, the `/var` hierarchy is for **variable data files**; those which change frequently or persist across operations such as logs, caches, mail spools, and runtime state. This separation enables `/usr` to remain static or mount read-only without loss of system functionality.

### Common `/var` Subdirectories

+ `/var/cache`; Application cache files.
+ `/var/lib`; Persistent state information changed by programs.
+ `/var/log`; System and application log files.
+ `/var/run`; Runtime process information (often symlinked to `/run`).
+ `/var/spool`; Queued/spooled data (e.g., mail, print queues).
+ `/var/tmp`; Temporary files preserved between reboots.
+ `/var/lock`; Lock files for resource control.
+ `/var/opt`; Variable data for `/opt` packages.

Special directories reserved due to historical or practical constraints include `/var/backups`, `/var/cron`, and `/var/preserve`.

**Compliance Implication:** New applications should not arbitrarily create top-level directories in `/var` without consideration of naming conflicts and standard conventions.

---

## Complementary and Virtual Filesystems

Although not specified as required by FHS, modern Linux systems often incorporate additional mount points not governed by the standard:

* `/proc`; Virtual filesystem exposing kernel and process information.
* `/sys`; Kernel device and system hierarchy exposed as files.
* `/dev/shm`, `/run`; Volatile memory filesystems.

These are provided by kernel subsystems and are integral to runtime behavior, but **are not mandated by FHS itself**.

---

## Practical Considerations and Best Practices

### Mounting Strategies

* Mount `/usr` read-only where possible; separate partitions can simplify maintenance.
* Isolate `/var` if log volumes or cache use is substantial.
* Use `/mnt` and `/media` consistently for removable or temporary mounts.

### Software Deployment

* System packages use `/usr/bin`, `/usr/lib`, etc., for binaries and libraries.
* Locally compiled or administratively installed applications should reside in `/usr/local`.

### Portability and Scripting

Scripts and configuration management tooling should reference directories expected under FHS to ensure cross-distribution compatibility.

### Containers and Virtualized Environments

Container images rely on predictable FHS layout (e.g., in Docker or Kubernetes) to ensure consistency across environments and orchestration workflows.

---

# Users and groups in Linux

## Overview

In Unix and Linux, a *user* is an abstract principal used to represent an actor (human, service, or process) to the operating system. That principal is identified to the kernel and user-space subsystems primarily by a numeric identifier: the **user identifier (UID)**. Analogously, a *group* is a named collection of principals represented by a **group identifier (GID)**. The system uses UIDs and GIDs (integers stored in process credentials and filesystem metadata) to make access-control and accounting decisions. Human-readable names (e.g., `alice`, `wheel`, `www-data`) are conveniences that map to those numeric identifiers via the system’s name service.

---

## Names versus numeric identifiers (why two layers)

There are always two layers in Linux identity:

* **Symbolic names** (strings) are what administrators and users interact with. They are stored in databases or services (local files, LDAP, NIS).
* **Numeric IDs** (UID/GID) are what the kernel and filesystems actually use for enforcement and storage.

This separation exists because kernel-level structures and on-disk formats are numeric for compactness and stability; name-to-number translation is provided by the Name Service Switch (NSS) and related infrastructure. The mapping is many-to-one in the sense that multiple names can, in theory, map to the same UID, but that breaks typical assumptions and is discouraged because enforcement is numeric.

---

## Process credentials: real, effective, saved (and filesystem) IDs

A running process carries credential fields that determine *who it acts as*:

* **Real UID / Real GID (ruid/rgid):** the identity of the actor that started the process (useful for accounting and identifying origin).
* **Effective UID / Effective GID (euid/egid):** the identity used for most access-control checks (file I/O, resource access).
* **Saved UID / Saved GID:** used to allow temporarily dropping and later restoring elevated privileges.
* **Supplementary group list:** additional GIDs that broaden group-based access.

Unix-derived systems use these to support controlled privilege elevation (for example, programs that run with owner privileges via setuid), delegation, and least-privilege patterns. Access checks are performed against the process’s effective IDs and its supplementary groups.

---

## Primary vs. supplementary groups; group semantics

Every user has a **primary group** (a single GID associated with the user) and may be a member of zero or more **supplementary groups**. The primary group often determines the group ownership applied to newly created files (behavior depends on filesystem and mount semantics), while supplementary groups provide additional access rights for group-owned resources. Groups are logical access-control domains: they exist to cluster principals for permission grants. Importantly, group membership is an attribute of a process at login (or when credentials are changed), not something the kernel looks up dynamically at each access.

---

## POSIX permission model: owner / group / other

The classical Unix permission model assigns three triads (owner, group, other) each with read, write, execute bits. File ownership is a pair (UID, GID) stored in filesystem metadata; the kernel compares the process credentials to those values to decide which triad applies, then enforces the permission bits.

This model is intentionally simple and forms the baseline for access control: it allows efficient enforcement but is coarse-grained. Modern Linux supports extensions (POSIX ACLs, extended attributes) and discretionary access-control frameworks (SELinux, AppArmor) when the three-way model is insufficient.

---

## Special permission bits and behavioral semantics

Beyond the basic permission bits, Unix defines special bits that alter execution or directory semantics:

* **setuid**: causes an executable to run with the file owner’s effective UID.
* **setgid**: for executables, analogous to setuid but for GID; for directories, it can cause new files/subdirectories to inherit group ownership from the directory.
* **sticky bit** (on directories): prevents users from removing or renaming other users’ files within the directory (common on world-writable directories like `/tmp`).

These mechanisms provide controlled privilege elevation and administrative semantics but introduce security risks if misapplied (e.g., setuid root binaries).

---

## Authentication vs. authorization; identity stores and PAM

Two distinct concerns must be conceptually separated:

* **Authentication** answers “who are you?” (validating credentials such as passwords, tokens, or Kerberos tickets).
* **Authorization** answers “what may you do?” (mapping identity to privileges and checking access-control rules).

Linux mediates authentication through pluggable frameworks such as PAM (Pluggable Authentication Modules), enabling flexible credential validation and integration with external identity providers (LDAP, Kerberos). Authorization is primarily enforced by kernel credentials and filesystem/security modules; centralized identity services provide consistent mappings of names to numeric IDs across machines in an environment.

---

## Centralized identity and distributed environments

In single-host deployments, the canonical name-to-number mapping is typically `/etc/passwd` and `/etc/group`. In enterprise contexts, identity is often centralized (LDAP, Active Directory via LDAP/Kerberos), and NSS/SSSD/CIFS components resolve names to IDs across many machines. When identities are shared across machines (for example, with NFS), consistency of UID/GID assignments becomes a critical architectural concern because enforcement is numeric and inconsistent mappings lead to misattributed access.

---

## System accounts vs. human accounts; reserved ranges

Operating systems distinguish *system/service accounts* (used by daemons and system components) from *regular human accounts*. Distributions reserve certain UID/GID ranges for static system accounts and for dynamically allocated system accounts; user-account ranges typically start at a specific threshold (e.g., 1000 on many modern distributions). The special UID `0` is the traditional superuser (root) and implicitly bypasses ordinary permission checks. System architects must plan UID/GID allocation schemes to avoid collisions in distributed environments.

---

## Namespaces and containerization: remapping identi

Linux introduces *user namespaces* that provide a mechanism to map UIDs/GIDs inside a container to different values outside the container. This allows processes to appear as root (UID 0) inside a namespace while being unprivileged externally—an important conceptual tool for containment and reduced host-surface risk. Namespaces change the semantic mapping between numeric IDs and privileges and are therefore a core concept when reasoning about identity in modern cloud and containerized deployments.

---

## Security considerations and common failure modes (conceptual)

* **Shared UIDs:** assigning the same UID to multiple named accounts removes logical separation of principals; auditing and fine-grained accountability are degraded.
* **Excessive setuid/setgid usage:** introduces escalation vectors if the privileged binary is exploitable.
* **Inconsistent UID/GID mappings across hosts:** causes privilege leaks or denial of access in networked filesystems.
* **Trust boundaries:** authentication (e.g., via Kerberos) does not guarantee authorization on the local host; local policy decisions and ACLs still determine access.

---

## Conceptual best practices (architectural guidelines)

* Treat numeric IDs as canonical; design name services so that name-to-number mappings are stable and centralized where needed.
* Enforce least privilege by minimizing use of privileged bits (setuid/setgid) and avoiding broad group grants.
* Segregate system/service UIDs from human user UIDs and reserve dedicated ranges for externally managed identities.
* Use namespaces and containerization to reduce host-level privilege exposure for service processes.
* Separate authentication and authorization responsibilities: centralize authentication where beneficial, but retain local authorization controls for final enforcement.

---

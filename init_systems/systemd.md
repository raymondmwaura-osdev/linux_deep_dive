# Introduction to systemd

`systemd` is the **modern init system and service manager for Linux operating systems**. It serves as the **first process launched by the kernel** (PID 1) and remains responsible for **managing system startup, services, and ongoing resource control** throughout system runtime. `systemd` is now the **default init system in major Linux distributions** due to its performance, scalability, and integrated service management capabilities.

---

## Purpose and Scope

At its core, `systemd` is designed to:

* **Initialize user space** and transition the system from kernel execution to a fully operational environment.
* **Manage system services and daemons** (start, stop, restart, supervise).
* **Coordinate dependencies and ordering** between services and system resources.
* **Provide integrated logging, resource control, and service activation mechanisms.**

`systemd` replaces legacy init systems such as SysVinit and Upstart by offering a **declarative, dependency-aware, and parallelized approach** to system initialization.

---

## Key Concepts

### PID 1: The First Process

During boot, the Linux kernel executes a single user-space process with **Process ID 1 (PID 1)**. In `systemd`-based systems, this process is the **systemd init manager** itself, making it the **root of all other processes**. It remains active until system shutdown and is responsible for how all subsequent processes are launched and supervised.

### Unit Files

`systemd` organizes managed entities into **units**, which are logical configuration objects that describe system resources (e.g., services, sockets, mounts). Each unit is backed by a **unit file**, typically stored under system directories such as:

* `/etc/systemd/system/`
* `/lib/systemd/system/`

A unit file defines metadata and instructions for how that unit should behave (start conditions, dependencies, execution commands).

### Targets

Targets replace the concept of runlevels from older init systems. A **target** is a unit that groups other units into a logical system state (for example, multi-user mode vs graphical mode). Targets allow administrators to transition the system state cohesively based on dependencies.

### Dependencies and Parallelization

`systemd` uses explicit dependency specifications (e.g., `Requires=`, `After=`) to build a **directed graph of service relationships**. This enables:

* **Parallel startup of independent services**, reducing boot time.
* **Ordered startup of dependent services** without manual scripting.

### Integrated Logging (journald)

Unlike legacy systems that rely on external syslog daemons, `systemd` includes **journald**, a structured logging subsystem that captures messages from the init system and all managed units. Administrators can analyze logs centrally using `journalctl`.

---

## Practical Commands

Once the system is running with `systemd`, administrators interact with it using tools such as:

### systemctl

`systemctl` is the primary command-line interface to control `systemd`. Common usage patterns include:

```bash
sudo systemctl start <unit>        # Start a service unit
sudo systemctl stop <unit>         # Stop a service unit
sudo systemctl status <unit>       # Show status and state
sudo systemctl enable <unit>       # Enable a unit at boot
sudo systemctl disable <unit>      # Disable a unit at boot
```

These commands provide uniform service and system state management across distributions.

### journalctl

`journalctl` allows querying and filtering `systemd` logs:

```bash
sudo journalctl -u <unit>          # View logs for a specific service
sudo journalctl --since "1 hour ago" # View logs since a timestamp
```

This eliminates the need to inspect multiple disparate log files.

---

## Why systemd Matters

Compared to legacy init systems:

* **Parallel service startup** significantly lowers boot times.
* **Dependency-aware scheduling** replaces brittle sequential scripts.
* **Centralized, structured logging** improves diagnostics.
* **Integrated resource management** enables sophisticated supervision.

These design choices reflect a shift toward **declarative system configuration, automated dependency handling, and performance optimization**; all aligned with modern multi-core and distributed infrastructures.

---

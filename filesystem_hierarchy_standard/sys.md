# The `/sys` Directory in Linux: A Deep Architectural Analysis

## Introduction

The `/sys` directory is a **virtual filesystem that exposes kernel objects, device attributes, and system topology** to user space. Known as **sysfs**, it is dynamically populated by the kernel and is essential for:

* Hardware introspection
* Kernel configuration and control
* User-space interaction with device drivers

From a system architecture perspective, `/sys` represents a **structured, hierarchical interface** between the kernel and user space for **runtime management and observability**.

---

## Historical Context

### Origins

* Introduced in Linux 2.5.x (early 2000s)
* Designed to complement `/proc` by providing **structured, attribute-oriented access**
* `/proc` primarily exposed process information and legacy kernel data
* `/sys` was created to represent **object-oriented kernel state**, including devices and buses

### Motivation

Before sysfs:

* Device attributes were scattered in `/proc` or handled via `ioctl`
* User-space lacked a consistent, discoverable hierarchy
* Dynamic devices (hot-plug, USB, PCI) were difficult to manage

Sysfs solved this by **formalizing kernel objects into a filesystem hierarchy**.

---

## `/sys` as a Virtual Filesystem

### tmpfs-backed, Ephemeral

* Mounted as a `sysfs` filesystem (not `tmpfs` per se, but in-memory and dynamic)
* Contents are **transient**, reflecting current kernel state
* Exists only while the system is running

### Hierarchical Organization

`/sys` represents kernel objects as **directories and files**:

* Directories = kernel objects (devices, buses, subsystems)
* Files = attributes (read/write values)
* Symbolic links = relationships (e.g., parent/child devices, drivers)

---

## Key Subsystems in `/sys`

### `/sys/class`

* Represents **device classes**, such as:

  * `net/` → network interfaces
  * `block/` → block devices
  * `tty/` → terminal devices
* Abstracts hardware type rather than physical location

### `/sys/block` and `/sys/devices`

* `/sys/block` → high-level block devices (sda, nvme0n1)
* `/sys/devices` → physical device tree (PCI, USB, platform devices)
* Enables **tracing logical devices to physical hardware**

### `/sys/bus`

* Represents bus subsystems (PCI, USB, platform, ACPI)
* Each bus exposes devices and drivers attached
* Facilitates hotplug and dynamic device enumeration

### `/sys/module`

* Exposes **loaded kernel modules** and their parameters
* Enables dynamic introspection and tuning

---

## Device Attribute Management

### Read/Write Semantics

* Attribute files are generally **text-based**
* Reading → returns current kernel value
* Writing → triggers kernel callbacks (if writable)
* Supports both monitoring and configuration of devices

### Examples

* `/sys/class/net/eth0/operstate` → current network interface state
* `/sys/class/backlight/intel_backlight/brightness` → adjust screen brightness
* `/sys/devices/system/cpu/cpu0/cpufreq/scaling_governor` → CPU governor control

---

## `/sys` vs `/proc`

| Aspect          | `/proc`                        | `/sys`                                      |
| --------------- | ------------------------------ | ------------------------------------------- |
| Primary purpose | Process info, kernel stats     | Kernel objects, device attributes           |
| Organization    | Mostly flat or semi-structured | Object-oriented, hierarchical               |
| Modifiability   | Limited writable interfaces    | Many writable attributes for device control |
| Persistence     | Transient, runtime state       | Transient, kernel runtime reflection        |

* `/sys` complements `/proc`, not replaces it
* `/sys` emphasizes **discoverability and structured representation**

---

## Security and Access Control

* Permissions mirror kernel policy
* Typically owned by root, with selective write access
* Misuse can affect device behavior (e.g., writing invalid values to power or frequency files)
* Interfaces are often exposed through udev rules or other user-space daemons rather than direct manual edits

---

## Use Cases and Applications

### Hardware Monitoring

* Tools like `lspci`, `lsusb`, `hwmon` query `/sys` for hardware state

### Device Management

* `udev` uses `/sys` events for hotplug detection and device node creation
* Allows dynamic naming, grouping, and permission assignment

### Runtime Tuning

* CPU frequency scaling, power management, backlight adjustment, thermal control

---

## Volatility and Ephemerality

* `/sys` is **not persistent across reboots**
* Directories and files are **kernel-generated at runtime**
* Reflects **current system configuration**, not stored configuration

This makes `/sys` conceptually similar to `/proc` and `/run` in that it **represents live system state**.

---

# Linux Installation Process: Internal System Stages

Linux installation is fundamentally a **system orchestration workflow** where firmware, disk subsystems, installer logic, bootloader, kernel, and configuration engines collaborate to transform raw hardware into an operational OS. This process can be conceptualized as a pipeline of discrete stages, each with specific responsibilities.

---

## 1. Firmware Initialization (Pre-Installer Execution)

**Objective:** Initialize hardware and prepare bootstrap environment.

**Mechanisms:**

* On x86-architecture systems, the **BIOS or UEFI firmware** executes at power-on/reset.
* Firmware performs a **Power-On Self Test (POST)** to initialize CPU, memory controllers, chipset, and I/O hardware.
* Firmware enumerates and initializes storage buses and devices, validating device presence and readiness.
* It then searches for bootable media (USB drive, CD/DVD, network PXE, etc.) and loads the **installer bootloader** into RAM.

**Internal Notes:**
UEFI firmware uses a **GUID Partition Table (GPT)** and the EFI System Partition to locate and execute EFI binaries. Legacy BIOS uses the Master Boot Record (MBR) to locate stage-1 bootstrap code.

---

## 2. Installer Bootloader Execution

**Objective:** Invoke the Linux installer kernel and initial environment.

**Mechanisms:**

* The firmware hands execution to a **bootloader on the installation media**, often a GRUB2 image or equivalent.
* The bootloader loads the installer’s **Linux kernel image** and associated **initial RAM filesystem (initramfs)** into memory.
* It passes **kernel parameters** that signal the boot-to-installer context (e.g., boot from live ISO, partition setup mode).

**Internal Notes:**
The initramfs is a temporary root filesystem that enables the kernel and early user-space utilities (including the installer environment) to run before the actual disk root file systems exist.

---

## 3. Installer Kernel and Environment Activation

**Objective:** Establish an isolated installation environment.

**Mechanisms:**

* The kernel decompresses itself into memory and initializes essential subsystems: memory management, scheduler, device drivers, and block devices.
* Device enumeration occurs (SATA, NVMe, USB), enabling the system to “see” disks and partitions.
* Initramfs scripts mount a **temporary pseudo-root** that contains the installer’s user space.
* The system launches the installer as a user-space process instead of a typical init system at this stage.

**Internal Notes:**
This stage effectively boots a **live OS environment** residing in RAM, from which the installer application runs. Many distributions package the live environment and the installer into the same initramfs.

---

## 4. Disk Partitioning and File System Preparation

**Objective:** Define persistent storage layout and file system semantics.

**Installer Actions:**

* The installer queries all detected **block devices**.
* It writes a **partition table** (GPT or MBR) according to the selected schema.
* For each partition, it performs **file system creation operations** (e.g., mkfs.ext4, mkfs.xfs, Btrfs formatting), which involve:

  * Superblock and inode table initialization
  * Allocation of block group metadata
  * Creation of root directory and file system structures
* On disk subsystems requiring logical volumes (LVM) or RAID, metadata areas are created and device maps defined.

**Internal Notes:**
These operations change low-level on-disk data structures so that the kernel can mount these partitions later as persistent file systems.

---

## 5. Installation Software Deployment

**Objective:** Populate the file system with distribution-specific operational software.

**Mechanisms:**

* The installer copies the **base system files** into the appropriate partitions:

  * `/boot` (kernel images, bootloader config)
  * `/etc` (configuration skeletons)
  * `/usr`, `/var`, `/bin`, `/lib` (system and library binaries)
* A **package manager backend** (e.g., RPM, dpkg, pacman) is engaged to:

  * **Resolve dependencies**
  * **Unpack packages**
  * **Configure service units and file templates**
* This entails invoking native package installation logic, extracting metadata, and adjusting system states.

**Internal Notes:**
Modern installers may support custom configurations (software profiles, desktop environments) that determine which packages are installed.

---

## 6. Bootloader Installation and Configuration

**Objective:** Plant a mechanism that enables the installed system to boot autonomously.

**Installer Actions:**

* Write a **bootloader image** onto the target disk’s boot region:

  * For BIOS: embed GRUB’s stage-1 in the MBR and stage-2 assets in `/boot/grub`.
  * For UEFI: deploy a `.efi` executable into the EFI System Partition.
* Generate a **bootloader configuration file** that:

  * Lists the installed kernel versions
  * Points to the correct root filesystem UUID or label
  * Defines default boot behavior

**Internal Notes:**
The installer must reconcile firmware mode (UEFI/BIOS) with the chosen partitioning scheme. Misalignment here is a common source of boot failures.

---

## 7. System Configuration Phase

**Objective:** Persist essential system identity and policies.

**Mechanisms:**

* Create or update configuration artifacts for:

  * Host identity (hostname, locale)
  * Network configuration (interfaces, DHCP/static settings)
  * Account credentials (root password, user accounts)
  * Service definitions and targets (systemd units or legacy init scripts)
* Write the **filesystem table (`/etc/fstab`)** to ensure persistent mount points on boot.

**Internal Notes:**
At this stage the system transitions from installer context to the intended runtime context, committing final templates and values to disk.

---

## 8. First Boot and Init Activation

**Objective:** Launch the installed OS outside the installer environment.

**Computer Actions:**

* On reboot, firmware invokes the bootloader installed in step 6.
* The bootloader loads the **installed kernel** and an initramfs appropriate for the installed system.
* The kernel mounts the root file system as defined in its parameters and transfers control to the installed **init system** (commonly `systemd` or compatible).

**Runtime Internals: Init Execution**

* The init system (PID 1) reads configuration (targets, units, services).
* It mounts remaining file systems, starts daemons, and brings the system to multi-user/graphical states.

---

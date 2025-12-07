# GRUB

## What is GRUB

* GRUB stands for **GNU GRand Unified Bootloader**.
* It is a boot loader and boot manager used primarily in Unix-like systems (Linux, BSD, GNU/Hurd, etc.), but capable of loading proprietary operating systems via “chain-loading”.
* The project is maintained by the GNU Project, and modern Linux distributions generally deploy the current major iteration, GRUB 2.

GRUB’s role is to serve as the intermediary between the firmware (BIOS or UEFI) and the operating-system kernel. At boot time, GRUB loads itself, presents a menu (if configured), lets the user choose which OS or kernel configuration to launch, then loads the selected kernel into memory and passes control.

Because of this design, GRUB supports multi-boot setups (multiple OSes on the same machine) and flexible boot-time customization such as selecting kernel parameters or recovery modes.

---

## How GRUB Fits into the Boot Process

To understand GRUB, it helps to see where it sits in the overall boot chain:

1. **Firmware initialization**; On power-up, the system firmware (BIOS or modern UEFI) performs hardware initialization and basic checks.
2. **Transfer to bootloader**; The firmware then transfers control to the bootloader located on the primary storage device (e.g. a disk). For BIOS systems, this is typically the Master Boot Record (MBR); for UEFI systems, this can be an EFI System Partition (ESP).
3. **GRUB loading**; GRUB takes over. It may present a menu to the user, or automatically boot a default entry after a timeout.
4. **Kernel and initramfs loading**; GRUB loads the selected kernel (and optionally an initramfs / initrd), passes needed parameters (e.g. root filesystem specification), and finally hands control to the kernel. The kernel then mounts filesystems, initializes devices, and launches the init process (e.g., systemd or another init system).

Thus, GRUB acts as a flexible bridge; replacing any hardwired bootloader logic with a programmable, configurable system.

---

## Key Features and Capabilities of GRUB

GRUB (especially GRUB 2) brings several advantages over simpler bootloaders:

* **Multi-OS support and chain-loading**: GRUB can boot a wide variety of operating systems; not only Linux, but also Windows, BSD, Solaris, etc., via chain-loading.
* **Filesystem flexibility**: GRUB understands many filesystems, including those used by modern Linux installations, LVM, RAID, different partition schemes (GPT), etc. This lets it locate kernels and initramfs without relying on hardcoded sector addresses.
* **Dynamic configuration and scripting**: GRUB 2 uses a modular and script-based configuration: you can define boot entries, custom kernel parameters, recovery options, and more.
* **Rescue / recovery modes**: If something goes wrong (kernel misconfiguration, corrupted initramfs, etc.), GRUB’s built-in shell (or “rescue prompt”) can help you manually load kernels or fix issues.
* **Support for modern architectures**: GRUB supports various CPU architectures beyond classic x86; including x86-64, ARM, RISC-V, and more.

---

## Installing and Configuring GRUB 2

Here is a typical workflow to install and configure GRUB 2 on a Linux system.

### Installation

* On a running Linux system, install the GRUB package (often named `grub` or `grub2`); many distros handle this automatically.
* Identify the target disk where you want GRUB installed (e.g. `/dev/sda`, `/dev/nvme0n1`, etc.) via tools like `lsblk` or `fdisk`.
* Execute the installation command. For BIOS-based disk, for example:

  ```bash
  sudo grub-install /dev/sda  
  ```
  Replace `/dev/sda` with your disk identifier.
* After installing, generate the GRUB configuration (boot menu, OS entries, etc.) using:

  ```bash
  sudo update-grub  
  ```

  or, depending on distribution / naming:

  ```bash
  sudo grub-mkconfig -o /boot/grub/grub.cfg  
  ```
* Reboot. On next start-up, GRUB should present its menu and allow you to boot.

> **Caution**: Installing GRUB to the wrong disk or partition may render the system unbootable. Always verify your target before proceeding, and back up important data.

### Configuration

* In GRUB 2, the primary configuration file that the bootloader reads at boot is typically `/boot/grub/grub.cfg` (or `/boot/grub2/grub.cfg`, depending on distro).
* **Important:** You generally should **not** manually edit `grub.cfg`, because it is auto-generated. Instead, you modify the “source” configuration; usually in `/etc/default/grub` and the script-based directory `/etc/grub.d/`.
* After making changes, you must re-run the configuration generation command (e.g. `update-grub` or `grub-mkconfig`) so that the new settings are applied.
* Through these settings you can:

  * Set the default boot entry and timeout
  * Add or remove kernel entries (or other OS entries)
  * Pass kernel parameters (e.g. specifying root filesystem, options like “quiet”, “ro”, “splash”, custom init, etc.)
  * Manage advanced setups such as multi-boot, LVM/RAID support, custom initramfs, recovery kernels, etc.

---

## GRUB in Dual-Boot / Multi-OS Environments

One of GRUB’s greatest strengths is enabling multi-boot configurations (e.g. Linux + Windows, or several Linux distributions).

* By scanning disks and partitions at install time (or whenever configuration is updated), GRUB can detect other operating systems and automatically add them to the boot menu.
* If automatic detection fails (for instance, if another OS is not on a standard partition), it is still possible to add manual entries; by editing the configuration scripts under `/etc/grub.d/` (or equivalent) and then regenerating `grub.cfg`.
* This flexibility makes GRUB ideal for developers, testers, or advanced users who maintain multiple OS installations; or even experimental OS kernels.

---

## GRUB Legacy vs GRUB 2

Historically, GRUB’s older version (often called GRUB Legacy) was simpler, but had limitations. The modern GRUB 2 offers significant improvements.

Key differences:

* GRUB 2 supports many more filesystems, LVM/RAID, GPT partitioning; legacy GRUB was more limited.
* The configuration scheme changed: GRUB Legacy used a static menu file (e.g. `menu.lst`), while GRUB 2 uses a generated configuration (`grub.cfg`) built from source files and scripts. This adds flexibility and reduces manual error.
* GRUB 2 includes modular and script-based architecture, support for themes, dynamic OS detection, and even a Bash-like rescue shell.

Because of these improvements, GRUB Legacy is now considered deprecated and GRUB 2 is the standard in nearly all modern Linux distributions.

---

## Example: Installing GRUB on a New System (BIOS / MBR)

Suppose you have a fresh Linux installation and want to install GRUB 2 to the main disk `/dev/sda`. Here is a minimal example of steps you might take:

```bash
# (as root)
# 1. Install GRUB package (distribution-specific; often already installed)
apt-get install grub   # or grub2, depending on distro

# 2. Identify target disk
lsblk
# Suppose your root disk is /dev/sda

# 3. Install GRUB to MBR of /dev/sda
grub-install /dev/sda

# 4. Generate GRUB configuration
update-grub              # or: grub-mkconfig -o /boot/grub/grub.cfg

# 5. Reboot
reboot
```

After reboot, the GRUB menu should appear (unless configured to skip it) and allow you to select the OS or kernel.

If your system uses UEFI rather than legacy BIOS, the process differs slightly: GRUB will install into the EFI System Partition, and an EFI-compatible binary such as `grubx64.efi` will be used.

---

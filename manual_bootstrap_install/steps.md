# 1. High-Level Strategy

You will:

1. Partition and format the target USB.
2. Bootstrap the Void Linux root filesystem directly into the USB using **xbps**.
3. Chroot into it to finish OS configuration.
4. Install GRUB to the USB so it boots independently.

This creates a fully portable Void Linux installation that behaves like a real disk install—no ISO flashing, no installers.

---

# 2. Install the Tooling You Need on Debian

You need `xbps` (Void’s package manager) available on Debian:

```bash
sudo apt update
sudo apt install xbps
```

If Debian’s repo doesn’t have a recent xbps, install it from Void’s GitHub releases, but usually Debian Stable includes it.

---

# 3. Identify and Partition the USB

Determine the device:

```bash
lsblk -p
```

Assume the target is `/dev/sdb`. Replace accordingly.

Create a clean GPT layout with boot + root partitions:

```bash
sudo parted /dev/sdb -- mklabel gpt
sudo parted /dev/sdb -- mkpart ESP fat32 1MiB 513MiB
sudo parted /dev/sdb -- set 1 esp on
sudo parted /dev/sdb -- mkpart rootfs ext4 513MiB 100%
```

Format:

```bash
sudo mkfs.vfat -F32 /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
```

Mount:

```bash
sudo mount /dev/sdb2 /mnt
sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sdb1 /mnt/boot/efi
```

---

# 4. Bootstrap a Void Linux Base System Directly Into /mnt

Use xbps to build a fresh root filesystem:

```bash
sudo xbps-install -S -R https://repo.voidlinux.org/current -r /mnt base-system
```

This is the core deliverable: a minimal, functional Void deployment staged at `/mnt`.

---

# 5. Bind-Mount System Resources for Chroot

This aligns the environment so you can finalize the system like it’s running natively:

```bash
sudo mount -t proc /proc /mnt/proc
sudo mount --rbind /sys /mnt/sys
sudo mount --rbind /dev /mnt/dev
sudo mount --rbind /run /mnt/run
```

Chroot:

```bash
sudo chroot /mnt /bin/bash
```

---

# 6. Inside the Chroot: Core System Configuration

Configure timezone, fstab, hostname, and a user.

**Fstab:**

```bash
blkid
```

Take the UUIDs and create `/etc/fstab`:

```
UUID=<root-uuid>    /           ext4    defaults        0 1
UUID=<efi-uuid>     /boot/efi   vfat    defaults        0 2
```

**Hostname:**

```bash
echo voidusb > /etc/hostname
```

**Timezone:**

```bash
ln -sf /usr/share/zoneinfo/UTC /etc/localtime
```

**User + password:**

```bash
passwd
useradd -m -G wheel username
passwd username
```

Enable sudo by editing `/etc/sudoers` if needed:

```bash
visudo
```

---

# 7. Install GRUB Directly to the USB Device

Critical: install to **/dev/sdb**, not /dev/sdb1.

```bash
xbps-install -y grub-x86_64-efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Void
grub-mkconfig -o /boot/grub/grub.cfg
```

If the target system uses BIOS/legacy, also run:

```bash
xbps-install -y grub-bios
grub-install /dev/sdb
```

---

# 8. Exit and Unmount

```bash
exit
sudo umount -R /mnt
sync
```

At this point, the USB contains a full, bootable Void Linux installation that will behave like a normal disk.

---

# 9. Operational Notes

* Booting on different hardware may require enabling Void’s `dracut` modules or ensuring the base system includes necessary filesystem/USB drivers.
* If system boots but network fails, ensure `dhcpcd` or `NetworkManager` is installed depending on your preference.
* If EFI boot insertion fails, temporarily disable Secure Boot or ensure the target firmware allows external EFI entries.

---

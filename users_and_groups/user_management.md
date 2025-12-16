# Introduction to Linux User Management

## Conceptual Overview

Linux is fundamentally a **multi-user operating system**. Effective user management is indispensable for system security, access control, and operational integrity. A robust user management strategy ensures that individual accounts have appropriate permissions and that administrative functions are traceable and auditable

User accounts in Linux represent discrete identities with unique **User Identifiers (UIDs)**, home directories, and associated groups. Linux distinguishes between:

* **Root (Superuser)**; Full system privileges.
* **Regular users**; Standard accounts with restricted privileges.
* **Sudo-enabled users**; Regular users granted temporary elevated privileges via `sudo`.
* **System/service accounts**; Non-interactive accounts used by system services

The user management model enforces **authentication** and **authorization** separately: authentication confirms identity (e.g., password verification), while authorization governs permitted actions

---

## Core System Files

Understanding the following configuration files is foundational:

* **`/etc/passwd`**; Contains basic user account metadata (username, UID, primary GID, home directory, default shell). Password hashes are *not* stored here
* **`/etc/shadow`**; Stores encrypted passwords and password aging policies; accessible only to privileged processes
* **`/etc/group`**; Defines groups and their member lists; groups facilitate collective permission assignments
* **`/etc/sudoers`**; Configures which users may execute commands with elevated privileges via `sudo`

These files underpin how Linux maps human users and roles to system permissions and access rights.

---

## Account Lifecycle Commands

### Creating Users

To establish a new user account:

```sh
sudo useradd -m -s /bin/bash username
```

* `-m`: instructs the system to create a home directory.
* `-s`: sets the user’s default login shell.
* `username`: the account identifier

Alternatively, distributions often provide `adduser`, a higher-level script that prompts interactively for password and user details. The two are functionally similar but differ in defaults and behavior

---

### Securing Accounts

Assign or update the user’s password with:

```sh
sudo passwd username
```

The `passwd` utility hashes the entered password and stores it in `/etc/shadow`. Password hashes are compared against login attempts for authentication

---

### Modifying User Accounts

Use the `usermod` command to adjust existing account properties:

* Add a user to supplementary groups:

  ```sh
  sudo usermod -aG groupname username
  ```

* Change a user’s home directory:

  ```sh
  sudo usermod -d /new/home/path username
  ```

* Change the login name:

  ```sh
  sudo usermod -l newname oldname
  ```

Modify UIDs, primary GIDs, and other metadata using respective `usermod` flags

---

### Deleting Users

To remove an account and optionally its home directory:

```sh
sudo userdel -r username
```

* `-r`: removes the user’s home directory and mail spool. Without this flag, only the account entry is removed

---

## Group Management Essentials

Groups allow administrators to assign permissions to a collection of users efficiently.

Common group commands:

* Create a group:

  ```sh
  sudo groupadd groupname
  ```

* Delete a group:

  ```sh
  sudo groupdel groupname
  ```

* Add a user to a group:

  ```sh
  sudo usermod -aG groupname username
  ```

Group membership affects access to shared resources and is a cornerstone of discretionary access control in Linux

---

## Querying and Auditing Accounts

Administrators frequently need to inspect existing accounts:

* List all users via `/etc/passwd`:

  ```sh
  awk -F: '{print $1}' /etc/passwd
  ```

* Inspect a user’s UIDs and groups:

  ```sh
  id username
  ```

These commands support verification of proper account configuration and assist in auditing

---

## Best Practices and Operational Considerations

* Prefer the use of `sudo` for administrative activity rather than logging in as root directly.
* Enforce strong password policies and consider password aging/expiration using tools like `chage`
* Organize users into groups that reflect role boundaries and access domains to enhance security and reduce complexity

---

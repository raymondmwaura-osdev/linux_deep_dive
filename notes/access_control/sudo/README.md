# `sudo`

## What is `sudo`?

* **`sudo`** is a command-line utility on Unix-like operating systems that permits a user to run a program with the security privileges of another user; by default, the superuser (root).
* The name “sudo” is commonly interpreted as “superuser do” (or “substitute user, do”).
* By design, `sudo` allows temporary elevation of privileges for specific commands, avoiding the need for a user to login as root for every administrative task.

---

## Why Use `sudo` Instead of Logging in as Root

Using `sudo` offers several advantages over simply logging in as root (e.g. via `su`):

* **Security and Principle of Least Privilege:** Regular users operate under normal permissions by default; `sudo` grants elevated privileges only when needed. This reduces the risk of accidentally performing system-level changes.
* **Accountability / Auditing:** Commands executed via `sudo` are logged (e.g., in system logs), providing a record of who ran what command and when.
* **Granular Control:** Rather than granting blanket root access, administrators can precisely define which users (or groups) can execute which commands, and under what conditions.

Because of these benefits, many Linux distributions favour `sudo` over enabling direct root login.

---

## Basic Usage of `sudo`

To use `sudo`, you typically prefix the command you wish to run with `sudo`. For example:

```bash
$ sudo apt update
```

When executed:

1. The system checks whether your user is authorized to run commands under `sudo` (via configuration).
2. If authorized, you’ll be prompted for *your* password; not the root password.
3. After authentication, the command runs with elevated privileges (as root, by default).

You can also run a command as another user (not root) by adding the `-u` option:

```
$ sudo -u otheruser some_command
```

If you omit `-u`, `sudo` defaults to root.

---

## Common `sudo` Options

`sudo` supports a variety of options for different use cases. Some common ones:

+ `-V`: Display version information of `sudo`.
+ `-h` / `--help`: Show usage/help message.
+ `-l`: List the commands the invoking user is allowed (or forbidden) to run.
+ `-s`: Run a shell with elevated privileges.
+ `-b`: Run the given command in the background.

---

## Configuring Who Can Use `sudo`; The `sudoers` File

* The permissions governing `sudo` usage are defined in a configuration file, typically `/etc/sudoers`.
* To safely edit this file, one should use the dedicated editor invoked via:

  ```
  sudo visudo
  ```

  This ensures syntax validation and prevents accidental corruption.

* Rather than granting blanket root-access to all users, an administrator can:

  * Allow specific users to run all commands.
  * Grant limited permissions to execute only certain commands.
  * Restrict the hosts, user identity, or environment under which commands run.

* Many distributions organize additional sudo rules in a directory such as `/etc/sudoers.d/`.

---

## Practical Example Workflow

1. **Check if `sudo` is available:**

   ```bash
   which sudo
   sudo -V
   ```

   If installed, `which sudo` typically returns `/usr/bin/sudo`.

2. **Run a command with elevated privileges:**

   ```bash
   sudo apt update
   sudo nano /etc/hosts
   ```

3. **List authorized commands (if you’re uncertain):**

   ```bash
   sudo -l
   ```

4. **If you administer the system, grant sudo access to a user:**

   ```bash
   # For Debian/Ubuntu-based distributions:
   sudo usermod -aG sudo username

   # For RedHat/CentOS-based distributions:
   sudo usermod -aG wheel username
   ```

   Then edit `/etc/sudoers` (via `visudo`) if custom permissions are required.

---

## Security Considerations and Best Practices

* Never give out the root password to ordinary users. With `sudo`, each user uses their own password, preserving secrecy and accountability.
* Limit `sudo` privileges; grant only the commands needed. Avoid giving full root access if unnecessary. Granular permissions reduce the blast radius of accidental or malicious misuse.
* Always use `visudo` for editing; syntax errors in `/etc/sudoers` may lock out administrative access.
* Maintain logs of `sudo` usage; this helps with auditing who made changes, when, and what was done.

---

# Linux Environment Variables

## Conceptual Foundations

### What Are Environment Variables

An **environment variable** is a **named key/value pair** maintained by the operating system (Linux/Unix) and inherited by processes at runtime. These variables define configuration values and contextual information that influence both **shell behavior** and **application execution** without modifying program source code. Each process has its own environment store, and by convention these variables are represented and accessed as strings

From a systems-level perspective:

* Environment variables are **inherited** from a process’s parent, typically the login shell or a supervisor process
* They play a critical role in **configuration management, runtime behavior, scripting, and security**.
* By convention, they are expressed in **uppercase identifiers** (e.g., `PATH`, `HOME`)

---

## Types of Variables

### Shell Variables vs. Environment Variables

* **Shell Variables:** Defined in the current shell session; they are **not exported** and therefore **not visible** to child processes.
* **Environment Variables:** Created by **exporting** a shell variable; they are **propagated to child processes** (e.g., scripts, daemons)

Example contrast:

```bash
name="temp"         # Shell variable (local to current shell)
export NAME="temp"  # Environment variable (visible to child processes)
```

---

## Common Predefined Environment Variables

These variables are typically initialized during login and maintained by shells such as **Bash**, **Zsh**, or **Dash**:

* **`PATH`**
  * Meaning: Search path for executables
  * Purpose: Defines the ordered list of directories the shell searches when resolving and executing commands.

* **`HOME`**
  * Meaning: User’s home directory
  * Purpose: Specifies the default directory for user files, configuration data, and application state.

* **`PWD`**
  * Meaning: Present working directory
  * Purpose: Maintains the absolute path of the shell’s current directory context.

* **`SHELL`**
  * Meaning: Default shell program
  * Purpose: Identifies the command interpreter used for interactive sessions and script execution.

* **`USER`**
  * Meaning: Username of the logged-in user
  * Purpose: Establishes user identity for processes, scripts, and access control decisions.

* **`LANG`**
  * Meaning: Locale and language settings
  * Purpose: Controls internationalization behavior, including character encoding, collation, and message language.

There are also less common, but operationally relevant, variables such as `LD_LIBRARY_PATH` (dynamic linker search path), `EDITOR` (default editor), and process configuration variables used by services and cron jobs

---

## Viewing and Inspecting Environment Variables

### Listing Variables

* **All environment variables:**

  ```bash
  printenv
  ```

  or

  ```bash
  env
  ```

  These commands emit key/value pairs representing the active environment

* **All variables including shell variables and functions:**

  ```bash
  set
  ```

  This is broader than `env` or `printenv`

### Querying a Specific Variable

To examine a specific environment variable:

```bash
echo $PATH
```

or

```bash
printenv HOME
```

Variable names are **case-sensitive** in Linux

---

## Setting Environment Variables

### Temporary Assignment (Session-Local)

To define a variable that **only affects the current shell session**:

```bash
VAR="value"
```

This will not be inherited by child processes.

### Exporting to Environment

To convert a shell variable into an environment variable:

```bash
export VAR="value"
```

This version persists for the life of the shell session and is inherited by subprocesses

### Modifying PATH

Appending a directory to `PATH` without replacing its existing contents:

```bash
export PATH="$PATH:/opt/bin"
```

This ensures the original search path remains intact

---

## Persistent Environment Variables

### User-Specific Persistence

To persist environment variables across sessions for a specific user:

1. Edit the user’s shell configuration file (e.g., `~/.bashrc`, `~/.profile`, or `~/.bash_profile` depending on shell).
2. Add `export` statements:

   ```bash
   export JAVA_HOME="/usr/lib/jvm/java-11"
   export PATH="$PATH:$JAVA_HOME/bin"
   ```
3. Source the file to apply immediately:

   ```bash
   source ~/.bashrc
   ```

Changes take effect for all new login shells

### System-Wide Persistence

For values that must apply to **all users**:

* Modify `/etc/environment`.
* Alternatively, create a script in `/etc/profile.d/`, which is sourced for all users.

Example `/etc/environment` entry:

```bash
DB_HOST="db.example.com"
```

System-wide changes frequently require **relogin or reboot** to take effect.([GeeksforGeeks][5])

---

## Unsetting and Cleaning Variables

To remove a variable from the environment in the current session:

```bash
unset VAR
```

Permanent removal requires editing configuration files (e.g., removing or commenting out the relevant `export` line)

---

## Operational Best Practices

* **Use descriptive names** that reflect purpose and context (e.g., `APP_CONFIG_DIR`).
* **Distinguish scopes**: temporary assignments for testing; exported variables for deployment contexts.
* **Audit and document variables** deployed in production, especially those containing credentials or endpoints.
* **Avoid accidental PATH overwrites** which can render system commands inaccessible.
* **Be aware of security exposure**: do not place sensitive values in world-readable configuration files

---

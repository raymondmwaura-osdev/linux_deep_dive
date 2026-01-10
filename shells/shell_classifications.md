# Introduction to Shell Classifications

In UNIX and UNIX‑like systems, the *shell* is the command‑line interpreter that mediates between the user and kernel services. In addition to providing a command syntax and execution environment, shells exhibit operational *modes* and *configurations* which affect their behavior, environment initialization, and usage context. At a high level, shells can be classified by:

* Their **operational mode** (interactive vs. non‑interactive)
* Their **session context** (login vs. non‑login)
* Their **security or capability constraints** (standard vs. restricted)

Each of these classifications has technical significance in system administration, scripting, environment initialization, and security policy.

---

# Operational Mode: Interactive vs. Non‑Interactive Shells

### Interactive Shells

An interactive shell is defined by its responsiveness to direct human input via a terminal interface. Such shells present prompts, accept commands, and return results in real time.

* **Definition:** A shell which *reads commands from a terminal (tty)* with input and output directed toward a user.
* **Characteristics:**

  * Presents a command prompt (e.g., `$PS1` in Bourne‑like shells).
  * Supports job control (background/foreground processes).
  * Maintains command history and completion facilities where supported by the implementation.
* **Typical Invocation:** Starting a shell manually in a terminal emulator (e.g., typing `bash`, `zsh`, etc.).
* **Use Cases:** System administration, exploratory development, interactive debugging.
* **Reference:** Shell interpreters operating in this mode read startup files and build an interactive environment accordingly.

### Non‑Interactive Shells

Non‑interactive shells operate without a human directly typing commands.

* **Definition:** A shell invoked to run commands from a *script, pipe, or an automated process* rather than from an interactive terminal session.
* **Characteristics:**

  * No prompt is displayed.
  * Standard input/output may be redirected to files or pipelines.
  * Typically used for automation and batch jobs.
  * Does not read the same set of initialization files as interactive shells unless explicitly configured.
* **Typical Invocation:** Executing a shell script (e.g., via `#!/bin/sh` shebang or automation tools).
* **Use Cases:** System startup scripts, cron jobs, deployment automation.
* **Reference:** Non‑interactive shells behave deterministically according to input files or commands without further user interaction.

---

# Session Context: Login vs. Non‑Login Shells

### Login Shells

A login shell is associated with the user’s authentication session.

* **Definition:** A shell that is started as part of a user *authentication event* (local console login, SSH session, or similar).
* **Technical Behavior:**

  * Processes login initialization files specific to the shell (e.g., `.bash_profile`, `.profile`, etc. in Bourne‑like shells).
  * Establishes environment variables and session context for the user.
  * Conventionally recognized via options such as `--login` in Bash.
* **Impact:** Determines which startup scripts are sourced and environment variables are exported.
* **Reference:** Login shells can be confirmed programmatically in Bash via shell options.

### Non‑Login Shells

A non‑login shell is initiated after the initial login environment is established.

* **Definition:** A shell not created as part of the session login procedure (e.g., starting a new terminal emulator after login).
* **Technical Behavior:**

  * Typically reads different initialization files (e.g., `.bashrc` for Bash).
  * Does not reprocess login time profiles, avoiding redundant environment setup.
* **Use Cases:** Multiple terminals within an authenticated session.
* **Reference:** Non‑login shells can still be interactive when attached to a terminal.

---

# Restricted Shells

### Concept and Purpose

Restricted shells impose limits on shell capabilities to enforce constrained environments, often for security or controlled access.

* **Definition:** A variant of a standard shell that *disables or inhibits specific features* of the shell language and environment.
* **Objective:** Reduce risk by preventing users from performing certain actions that could escalate privileges or circumvent controls.

### Behavior and Restrictions

Depending on the implementation (e.g., restricted Bash; `rbash`), a restricted shell may prohibit:

* Changing the working directory (`cd`).
* Modifying environment variables like `PATH`, `SHELL`, or `ENV`.
* Running commands via explicit pathnames.
* Using redirection or executing commands that could alter the environment.
* Extensive manipulation of shell configuration.

These restrictions are enforced after initialization and cannot be disabled during that session.
*Use Case:* Environments where users should be limited to a fixed set of operations or commands (e.g., kiosk accounts, shared systems, regulated consoles).

---

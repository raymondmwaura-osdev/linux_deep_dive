# Login

This file explains what program(s) is running when you are logging in to your user account on Linux.

## Graphical Login: A Display Manager

+ If you see a graphical screen with a welcome message and prompts for your username and password, that program is a **display manager**.
+ Examples of display managers:
    * **GDM (GNOME Display Manager)**: The default for GNOME, used by Ubuntu, Fedora, and others .
    * **SDDM (Simple Desktop Display Manager)**: The default for KDE Plasma, also used by LXQt .
    * **LightDM**: A cross-desktop display manager used by Ubuntu (for Xubuntu, Lubuntu), Linux Mint, and others .
    * **XDM (X Display Manager)**: An older, classic display manager .
    * **LXDM**: A lightweight display manager often used with LXDE .
    * **SLiM (Simple Login Manager)**: Another lightweight option, though now largely abandoned .

## Text-Based Login: Getty and Login

+ If instead of a graphical screen you see a terminal-like screen with a prompt asking for a login name, then the login process is handled by two programs working together:

1.  **`getty`**: This program (often `agetty` or `mingetty`) is responsible for displaying the initial `login:` prompt on the terminal.
2.  **`login`**: After you enter your username, `getty` passes control to the `login` program. `login` then prompts you for your password, checks it against the system's user database (usually `/etc/passwd` and `/etc/shadow`), and, if correct, starts your command-line shell.

---

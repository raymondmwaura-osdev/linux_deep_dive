# Display Manager

A display server is the first program you interact with when you turn on your computer and see a graphical login screen.

## The Purpose of a Display Manager

Its primary purpose is to manage user sessions. It acts as the bridge between the system's underlying graphical framework and your personal desktop.

*   **User Authentication**: It's the program that securely prompts you for your username and password, verifying your identity before you can access the system.
*   **Session Management**: A key feature is allowing you to **choose your desktop environment** (like GNOME, KDE Plasma, or XFCE) at the login screen. The display manager looks for installed desktop environments (usually in `/usr/share/xsessions`) and presents them as options. This allows you to switch between different environments without restarting your computer.
*   **Starting the Graphical Server**: It is responsible for launching the **display server** (like X11 or Wayland), which is the foundational software that draws everything on your screen and manages input from your keyboard and mouse. Without the display manager, you would have to start the graphical system manually from the command line, often using a command like `startx`.

## Overview of How It Does It

Here's a simplified view of how a display manager works.

1.  **Initialization**: When your computer boots, the system starts the display manager service (for example, the `gdm3` service for GNOME).
2.  **Starting the Server**: The display manager then starts the display server (e.g., an Xorg server) and manages it.
3.  **The Greeter**: It launches a graphical program known as the **"greeter"** (e.g., `gdmlogin`). This is the actual login window you see. This greeter displays options for users and desktop environments.
4.  **Authentication**: When you enter your credentials, the display manager uses a system like **PAM (Pluggable Authentication Modules)** to securely check your password against the system's user database.
5.  **Launching the Session**: Upon successful login, it reads the configuration for your chosen desktop environment. It then runs startup scripts (like those in `/etc/gdm3/PreSession/`) and finally launches the session, handing over control to your desktop environment. It also manages the environment, setting important variables like `XAUTHORITY` for secure access to the graphical server.

## A Note on Remote Access (XDMCP)

Historically, display managers also supported remote logins via a protocol called **XDMCP (X Display Manager Control Protocol)**. This allowed a user's computer to act as a "thin client" and display a login screen served by a more powerful machine on the network. While less common on home systems today, this feature highlights the display manager's role in managing both local and remote graphical sessions.

## Key Distinction: Display Manager vs. Display Server vs. Window Manager

* **Display Manager**: The login screen and session manager.
* **Display Server**: The software that manages the actual drawing on the screen and handles input from your keyboard/mouse. Examples include X11 (or Xorg) and Wayland.
* **Window Manager**: Software that controls the placement, appearance, and behavior of application windows (like borders, title bars, and how to minimize them). It is often part of or tightly integrated with a desktop environment, but it can also be a standalone program.

---

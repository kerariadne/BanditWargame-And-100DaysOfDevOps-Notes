# Welcome to the KodeKloud 100 Days of DevOps Challenge!  
Here you'll find daily tasks and progress updates as I work through the program.

## Day 1: Linux User Setup with Non-Interactive Shell

### What is a Non-Interactive Shell?

A **non-interactive shell** is a shell that is not connected to a terminal or user input. It runs scripts or commands automatically and the user does not interact with it.

### Why Use Non-Interactive Shell Users?

- **Limited-purpose users** that should not be able to log in interactively (via SSH or console)
- Useful for users tied to services (`nginx`, `mysql`, `www-data`) that don't need shell access and often run under their own users

### Commands for Creating Non-Interactive Users:

```bash
thor@jump_host ~$ sudo useradd -s /sbin/nologin user
# or
thor@jump_host ~$ sudo useradd -s /usr/sbin/nologin user  
# or
thor@jump_host ~$ sudo useradd -s /bin/false user
```

> **Note about `sudo`:**  
> `sudo` allows permitted users to run commands as another user, most commonly as root.  
> - Access is controlled by the `/etc/sudoers` file (and files in `/etc/sudoers.d/`).
> - `sudo` does **not** remove or disable the root account; it simply lets selected users temporarily act as root, usually without needing the root password.
> - By default, users authenticate with their own password (not root's).
> - This provides a secure way to grant temporary administrative privileges.

### Shell Options Explained (just for fun):

| Shell | Description |
|-------|-------------|
| `/sbin/nologin` | Blocks login and shows a message (same as `/usr/sbin/nologin`) |
| `/bin/false` | Immediately exits with failure (no message) |


### Login vs Non-Login Shells

The difference between **login** and **non-login** shells in Linux is how the shell is started and what initialization files are sourced (executed). This affects how your environment is set up, such as PATH, aliases, functions, and variables.

### Comparison Table: Login vs Non-Login Shells

| Type            | How it Starts                                 | Loads Files                                 | Purpose                        |
|-----------------|-----------------------------------------------|---------------------------------------------|--------------------------------|
| **Login Shell**     | SSH, TTY, `bash --login`                       | `/etc/profile`, `~/.bash_profile`, etc.     | Sets up session-wide environment variables and settings |
| **Non-Login Shell** | Terminal emulator, script, `bash`              | `~/.bashrc`                                 | Used for interactive commands, aliases, and functions   |


### Why Does It Matter?

When the `PATH` environment variable differs between shells, unexpected behavior can occur:

- Settings added to `~/.bashrc` may not be available during SSH sessions (login shells).
- Configurations in `~/.profile` might not apply to new terminal windows (non-login shells).

Understanding which initialization files are sourced by each shell type ensures environment variables and settings are consistently applied.

---

## Day 2: Temporary User Setup with Expiry

### Task: Create a temporary user named steve on App Server 3 with expiry date 2023-05-12

```bash
thor@jump_host ~$ ssh banner@stapp03
thor@jump_host ~$ sudo adduser -e 2023-05-12 steve
thor@jump_host ~$ sudo chage -l steve
```

### Commands Reference:

| Command | Description |
|---------|-------------|
| `adduser -e YYYY-MM-DD` | Specifies the expiration date in the format year-month-day |
| `chage -l` | Displays detailed information about password expiration and aging policies for a specified user |
| `chage -E` | Can also set expiry date to user |
| `passwd -l` | Disable password authentication |

### Why Do We Need Temporary Users?

External personnel may require access to a system for a specific project or duration. We may want to reduce attack surface on a system. Reasons can be many, but we can maintain an audit trail of who accessed the system and for how long.

### Audit Commands (for fun):

| Command | Description |
|---------|-------------|
| `last` | Shows the login history of users from `/var/log/wtmp` |
| `lastb` | Shows bad login attempts from `/var/log/btmp` |
| `w` | Shows who is logged in and what they are doing, including session duration |

---
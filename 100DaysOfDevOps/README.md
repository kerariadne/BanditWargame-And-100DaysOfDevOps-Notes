# <font color="#bd4a55">Welcome to the KodeKloud 100 Days of DevOps Challenge!🚀</font>
Here you'll find daily tasks and progress updates as I work through the program.

## Day 1: Linux User Setup with Non-Interactive Shell

### What is a Non-Interactive Shell?

A <font color="#c7675d">**non-interactive shell**</font> is a shell that is not connected to a terminal or user input. It runs scripts or commands automatically and the user does not interact with it.

### Why Use Non-Interactive Shell Users?

- <font color="#bb6569">**Limited-purpose users**</font> that should not be able to log in interactively (via SSH or console)
- Useful for users tied to services (`nginx`, `mysql`, `www-data`) that <font color="#0B67A0">don't need shell access</font> and often run under their own users

### Commands for Creating Non-Interactive Users:

```bash
thor@jump_host ~$ sudo useradd -s /sbin/nologin user
# or
thor@jump_host ~$ sudo useradd -s /usr/sbin/nologin user  
# or
thor@jump_host ~$ sudo useradd -s /bin/false user
```


>  <span style="color: #2E8B57; font-weight: bold;" > **Note about `sudo`:** </span> 
>- `sudo` allows permitted users to run commands as another user, most commonly as <span style="color: #DC143C;">root</span>.  
> - Access is controlled by the `/etc/sudoers` file (and files in `/etc/sudoers.d/`).
> - `sudo` does **not** remove or disable the root account; it simply lets selected users temporarily act as root, usually without needing the root password.
> - By default, users authenticate with their own password (not root's).
> - This provides a <span style="color: #2E8B57; font-weight: bold;" >secure way to grant temporary administrative privileges</span>.

### Shell Options Explained (just for fun⭐):

| Shell | Security Level | Description |
|-------|----------------|-------------|
| `/sbin/nologin` | <span style="color: #00855B;">**Secure**</span> | Blocks login and shows a message (same as `/usr/sbin/nologin`) |
| `/bin/false` | <span style="color: #ae8e7e;">**Moderate**</span> | Immediately exits with failure (no message) |


### Login vs Non-Login Shells

The difference between **login** and **non-login** shells in Linux is how the shell is started and what initialization files are sourced (executed). This affects how our environment is set up, such as PATH, aliases, functions, and variables.

### Comparison Table: Login vs Non-Login Shells

| Type            | Priority | How it Starts                                 | Loads Files                                 | Purpose                        |
|-----------------|----------|-----------------------------------------------|---------------------------------------------|--------------------------------|
| **Login Shell**     | <span style="color: #00855B;">**High**</span>  | SSH, TTY, `bash --login`                       | `/etc/profile`, `~/.bash_profile`, etc.     | Sets up session-wide environment variables and settings |
| **Non-Login Shell** | <span style="color: #ae8e7e;">**Medium**</span> | Terminal emulator, script, `bash`              | `~/.bashrc`                                 | Used for interactive commands, aliases, and functions   |


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

| Command | Priority | Description |
|---------|----------|-------------|
| `adduser -e YYYY-MM-DD` | <span style="color: #2E8B57;">**Essential**</span> | Specifies the expiration date in the format year-month-day |
| `chage -l` | <span style="color: #DAA520;">**Useful**</span> | Displays detailed information about password expiration and aging policies for a specified user |
| `chage -E` | <span style="color: #DAA520;">**Useful**</span> | Can also set expiry date to user |
| `passwd -l` | <span style="color: #DC143C;">**Critical**</span> | Disable password authentication |

### Why Do We Need Temporary Users?

<span style="color: #2E8B57;">External personnel may require access to a system for a specific project or duration.</span> We may want to reduce attack surface on a system. Reasons can be many, but we can maintain an <span style="color: #4169E1;">audit trail</span> of who accessed the system and for how long.

### Audit Commands (for fun):

| Command | Description |
|---------|-------------|
| `last` | Shows the login history of users from `/var/log/wtmp` |
| `lastb` | Shows bad login attempts from `/var/log/btmp` |
| `w` | Shows who is logged in and what they are doing, including session duration |

---

## Day 3: Secure Root SSH Access 

Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including the restriction of direct root SSH login. Your task is to disable direct SSH root login on all app servers within the Stratos Datacenter.


```bash
sudo vi /etc/ssh/sshd_config
    PermitRootLogin no
sudo systemctl restart sshd
```

> **Note:** Before disabling **root login**, we have to ensure that **we have at least one non-root user with sudo privileges configured** to access the server and perform administrative tasks!

### First of all why do we need to disable direct SSH root login? 💡
```diff
- If someone is able to find the root password or key via SSH, then they will have complete control over the system;
- If all operators log in as root at the same time, we won't be able to tell who did what :/
```
### Main options in sshd_config file:

- <font color="teal">PasswordAuthentication</font> – Enables or disables SSH logins using passwords.
- <font color="teal">PermitRootLogin</font> – Controls whether the root user can log in via SSH.
- <font color="teal">PermitEmptyPasswords</font> – If set to yes, allows accounts with no password to log in <font color="E3192A">(unsafe!)</font>
- <font color="teal">AllowUsers</font> or <font color="teal">DenyUsers</font> – Allow or deny specific users from logging in via SSH
- <font color="teal">AuthorizedKeysFile</font> – Specifies where to find a user’s public keys (default is .ssh/authorized_keys)
- <font color="teal">ChallengeResponseAuthentication</font> – For two-factor or challenge-based login

### Difference between <font color="FD647C">sshd_config</font>, <font color="CA0515">sshd_config.d/</font>, <font color="AB0007"> ssh_config</font>, and <font color="E3192A">ssh_config.d/</font>

- `/etc/ssh/sshd_config` is the main configuration file for the SSH server.
- `/etc/ssh/sshd_config.d/` is a directory for additional config files - anything placed here overrides the main sshd_config settings if there’s a conflict.

`/etc/ssh/ssh_config` and `/etc/ssh/ssh_config.d/` are for the SSH client - they control how our system behaves when connecting to other servers, not how our server accepts connections.

### Conflicting PasswordAuthentication settings

If we have two files like:

- `50-cloud-init.conf` with <span style="color: #DAA520;">`PasswordAuthentication yes`</span>
- `60-cloudimg-settings.conf` with <span style="color: #DC143C;">`PasswordAuthentication no`</span>

Then **second one** wins, because files inside sshd_config.d/ are loaded in **alphabetical order**, and **later files override earlier ones**.  
So in this case, <span style="color: #2E8B57;">`PasswordAuthentication no` is the one that takes effect — password-based logins are disabled.</span>

---

## Day 4: Linux File Permissions

Your task is to grant executable permissions to the `/tmp/xfusioncorp.sh` script on App Server 2. Additionally, ensure that all users have the capability to execute it.

```bash
sudo chmod a+x /tmp/xfusioncorp.sh
```
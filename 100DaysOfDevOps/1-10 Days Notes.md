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

---
## Day 5: SELinux Package Installation & Temporary Disable

### Task Overview

- **Install required SELinux packages.**
- **Permanently disable SELinux (to be re-enabled after configuration changes).**
- **No reboot needed now; maintenance reboot is scheduled for tonight.**
- **Ignore current SELinux status via CLI; after reboot, SELinux should be disabled.**

### Commands

```bash
# Install SELinux and related utilities
sudo yum -y install selinux* policycoreutils libselinux-utils setroubleshoot-server setools setools-console mcstrans

# Permanently disable SELinux
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

# Verify SELINUX setting in config file
cat /etc/sysconfig/selinux | grep SELINUX

# Check current SELinux status (will be disabled after reboot)
sudo sestatus
sudo getenforce

# Temporarily set SELinux to permissive mode (until reboot)
sudo setenforce 0
```
---

## Day 6: Create a Cron Job

- Install cronie package on all Nautilus app servers and start crond service
- Add a cron job `*/5 * * * * echo hello > /tmp/cron_text` for root user

```bash
# 1. Install cronie (for CentOS/RHEL)
sudo yum install -y cronie

# 2. Enable crond service to start at boot
sudo systemctl enable crond

# 3. Start crond service immediately
sudo systemctl start crond

# 4. Check status (optional, just to confirm)
sudo systemctl status crond

# 5. Edit root's crontab
sudo crontab -e

# 6. Add this line to the crontab:
*/5 * * * * echo hello > /tmp/cron_text

# 7. Verify cron job was added
sudo crontab -l
```

### Cron Schedule Examples Reference:

| Description | Cron Expression | Frequency |
|-------------|----------------|-----------|
| Run a script every minute | `* * * * *` | Every minute |
| Run at 15th minute of every hour | `15 * * * *` | Hourly at :15 |
| Run at 3 AM every day | `0 3 * * *` | Daily at 3:00 AM |
| Run on 10th of every month at midnight | `0 0 10 * *` | Monthly on 10th |
| Run every January 1st at midnight | `0 0 1 1 *` | Yearly on Jan 1st |
| Run at 8 AM every Monday | `0 8 * * 1` | Weekly on Monday |
| Run every 5 minutes | `*/5 * * * *` | Every 5 minutes |
| Run every 5 hours | `0 */5 * * *` | Every 5 hours |
| Run at 12:30 PM daily | `30 12 * * *` | Daily at 12:30 PM |
| Run weekdays at 9 AM | `0 9 * * 1-5` | Monday-Friday |
| Run first 10 minutes of each hour | `0-10 * * * *` | Minutes 0-10 |
| Run every 2 hours on 1st & 15th | `0 */2 1,15 * *` | Bi-monthly |
| Run every 10 min on Sundays in Dec | `*/10 * * 12 0` | December Sundays |
| Run at 6:45 PM on Fridays | `45 18 * * 5` | Friday evenings |

---

## Day 7: Linux SSH Authentication

The system admins team of xFusionCorp Industries has set up some scripts on jump host that run on regular intervals and perform operations on all app servers in Stratos Datacenter. To make these scripts work properly we need to make sure the `thor` user on jump host has password-less SSH access to all app servers through their respective sudo users. Set up password-less authentication from user `thor` on jump host to all app servers through their respective sudo users.

### How SSH Key Authentication Works 🔐

Password-less authentication in SSH works by using key pairs instead of passwords:

1. **Private key** stays on the jump host in thor's home directory — it's like your personal "secret signature"
2. **Public key** gets copied to the target server's `~/.ssh/authorized_keys` file for the sudo user (like tony)
3. When thor connects, SSH uses the private key to prove identity, and the target server checks the matching public key — no password prompt needed

### Why Generate SSH Keys?

If we skip key generation:
- ❌ we won't have a key pair for thor
- ❌ `ssh-copy-id` won't have a public key to install on the app servers
- ❌ SSH will still fall back to asking for a password
- ✅ It's basically creating the "digital handshake" so thor can log in automatically

### Step-by-Step Commands:

```bash
# 1. Generate SSH key pair on jump host (as thor user)
ssh-keygen -t rsa

# This will create:
# Public key: ~/.ssh/id_rsa.pub
# Private key: ~/.ssh/id_rsa

# 2. Copy the public key to each app server's sudo user
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03

# 3. Verify password-less login works
ssh tony@stapp01 "hostname"
ssh steve@stapp02 "hostname"
ssh banner@stapp03 "hostname"
```

### Key Files Created:

| File | Location | Purpose |
|------|----------|---------|
| `id_rsa` | `~/.ssh/id_rsa` | **Private Key** (Keep secret!) |
| `id_rsa.pub` | `~/.ssh/id_rsa.pub` | **Public Key** (Safe to share) |
| `authorized_keys` | Remote: `~/.ssh/authorized_keys` | Contains public keys for authentication |

---

## Day 8: Install Ansible

Install Ansible v4.8.0 on the jump host using pip3 and make sure it's available for all users.

### Installation Steps:

```bash
# Check OS version
cat /etc/*release

# Check if Python 3 is installed
python3 --version

# Check if pip3 is installed
pip3 --version

# Install Ansible using pip3
sudo pip3 install ansible==4.10.0

# Verify installation
ansible --version

# Check if it is available to all user (it should be located in /usr/local/bin)
which ansible

```

---

Day 8: Install Ansible

Install Ansible v4.8.0 on the jump host using pip3 and make sure it’s available for all users.



cat /etc/*release # to check OS
python3 --version #Check if Python 3 is installed
pip3 --version # to check if pip3 is installed
sudo pip3 install ansible==4.10.0

ansible --version

“core 2.11.12.” . This is because Ansible’s version command shows the core engine version, not the full package version. To check the full release, I had to run pip3 show ansible.

Since Ansible was installed with sudo pip3, It should be in /usr/local/bin and available to all users.
I checked with the command which ansible to confirm

---

## Day 9: MariaDB Troubleshooting

### Problem Statement

There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that **MariaDB service is down** on the database server.

**Task:** Look into the issue and fix the same.

### Troubleshooting Steps & Analysis

#### 1. Check Service Status

```bash
systemctl status mariadb.service
```

**Output:**
```
○ mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: inactive (dead) since Fri 2025-08-15 15:37:29 UTC; 9min ago
```

#### 2. Check MariaDB Logs

```bash
sudo cat /var/log/mariadb/mariadb.log
```

**Error Found:**
```
2025-08-15 15:53:22 0 [ERROR] mariadbd: Can't create/write to file '/run/mariadb/mariadb.pid' (Errcode: 13 "Permission denied")
2025-08-15 15:53:22 0 [ERROR] Can't start server: can't create PID file: Permission denied
```

#### 3. Investigate Directory Permissions

```bash
ls -ld /run/mariadb/
```

**Initial Output:**
```
drwxr-xr-x  2 root mysql    40 Aug 15 15:37 mariadb
```

**Root Cause:** The `/run/mariadb/` directory was owned by `root:mysql` instead of `mysql:mysql`, causing permission issues when MariaDB tried to create its PID file.

### Solution & Fix

#### 4. Fix Directory Ownership

```bash
sudo chown mysql:mysql /run/mariadb/
```

#### 5. Verify Ownership Change

```bash
ls -ld /run/mariadb/
```

**After Fix:**
```
drwxr-xr-x 2 mysql mysql 60 Aug 15 15:56 /run/mariadb/
```

#### 6. Start MariaDB Service

```bash
sudo systemctl start mariadb.service
```

#### 7. Verify Service is Running

```bash
sudo systemctl status mariadb.service
```

---

---

## Day 10: Bash Script for Website Backup

### Task: Create Backup Script for Static Website

Create a bash script for taking websites backup. They have a static website running on App Server 3 in Stratos Datacenter, and they need to create a bash script named `blog_backup.sh` which should accomplish the following tasks. (Also remember to place the script under `/scripts` directory on App Server 3).

**Requirements:**
- a. Create a zip archive named `xfusioncorp_blog.zip` of `/var/www/html/blog` directory
- b. Save the archive in `/backup/` on App Server 3. This is a temporary storage, as backups from this location will be clean on weekly basis. Therefore, we also need to save this backup archive on Nautilus Backup Server
- c. Copy the created archive to Nautilus Backup Server server in `/backup/` location
- d. Please make sure script won't ask for password while copying the archive file. Additionally, the respective server user (for example, tony in case of App Server 1) must be able to run it

### Setup Commands:

```bash
# Generate SSH key for passwordless authentication
ssh-keygen -t rsa 

# Copy SSH key to backup server
ssh-copy-id clint@stbkp01

# Make script executable
chmod +x /scripts/blog_backup.sh
```

### Script Content (`/scripts/blog_backup.sh`):

```bash
#!/bin/bash

# Create zip archive of the blog directory
zip -r /backup/xfusioncorp_blog.zip /var/www/html/blog

# Copy archive to Nautilus Backup Server
scp /backup/xfusioncorp_blog.zip clint@stbkp01:/backup/
```

...







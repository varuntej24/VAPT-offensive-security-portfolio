# Linux Fundamentals & Systems Administration (VAPT Reference)

## Overview
This technical reference consolidates core Linux operating system mechanics across basic filesystem navigation, user permissions, process management, automated system scheduling (cron), system services (systemd), and security log auditing. Mastering these controls is essential for host-level penetration testing and privilege escalation.

---

## 1. Linux Fundamentals (Parts 1 & 2 Core Synthesis)

### Filesystem Navigation & File Operations
* `/` (Root directory): The top-level hierarchy of the Linux filesystem.
* `/bin` & `/usr/bin`: Core executable binaries (commands).
* `/etc`: System-wide configuration files (e.g., `/etc/passwd`, `/etc/shadow`, `/etc/crontab`).
* `/var/log`: System and application event logs.
* `/home` & `/root`: User and administrative home directories.
* `/tmp` & `/dev/shm`: Temporary, world-writable directories used for staging exploits.

```bash
pwd                       # Print current working directory
ls -la                    # List all files including hidden (.) with detailed permissions
cd /var/log               # Change directory
cat file.txt              # Print contents of a file
less /var/log/syslog      # View file contents page-by-page (q to exit)
head -n 10 file.txt       # Display first 10 lines
tail -n 10 file.txt       # Display last 10 lines
tail -f /var/log/auth.log # Follow log updates in real-time
touch exploit.sh          # Create an empty file
mkdir -p /tmp/recon/tools # Create nested directories
cp source.txt dest.txt    # Copy file
mv old.txt /tmp/new.txt   # Move or rename file
rm -rf /tmp/junk          # Recursively and forcefully delete files/directories
```

### Operators & Pipeline Redirection
```bash
command > file.txt        # Overwrite stdout to file
command >> file.txt       # Append stdout to file
command 2> /dev/null      # Redirect stderr to null (silence permission denied errors)
cat file.txt | grep -i "user"  # Pipe stdout of command 1 into stdin of command 2
cat file.txt | wc -l      # Count lines in output
```

### Linux Permissions & Ownership Model
Permissions are divided into three tiers: **Owner (u)**, **Group (g)**, and **Others (o)**.

| Permission | Symbol | Octal Value | File Meaning | Directory Meaning |
| :---: | :---: | :---: | :--- | :--- |
| **Read** | `r` | `4` | View contents | List files inside (`ls`) |
| **Write** | `w` | `2` | Modify contents | Create/delete files inside |
| **Execute** | `x` | `1` | Run as script/binary | Enter directory (`cd`) |

```bash
# Octal permission syntax:
chmod 755 script.sh       # rwxr-xr-x (Owner full, Group/Others read+execute)
chmod 600 id_rsa          # rw------- (Owner read+write, required for SSH private keys)
chmod +x exploit.sh       # Add execution permission for all

# Changing ownership:
sudo chown user:group file.txt
sudo chown -R www-data:www-data /var/www/html
```

---

## 2. Linux Fundamentals: Part 3 Deep Dive (Task-by-Task Breakdown)

### Task 1: Deploy Your Machine
* **What it is:** Establishing your target host environment via TryHackMe AttackBox or OpenVPN connection.
* **Why do it:** Penetration testing requires an isolated target host to execute commands and verify privilege states.
* **How to do it:** Connect via OpenVPN (`sudo openvpn client.ovpn`) or launch the AttackBox, then SSH into the deployed IP:
  ```bash
  ssh <username>@<TARGET_IP>
  ```

---

### Task 2: Advanced Text Manipulation & Searching
* **What it is:** Advanced CLI tools to search for strings, extract patterns, and locate sensitive files across the entire filesystem.
* **Why do it:** Attackers and analysts need to hunt for passwords, private keys, API tokens, and misconfigured files inside thousands of system files.
* **How to do it:**
  ```bash
  # Search for text inside files:
  grep -rnwi "password" /etc/ 2>/dev/null
  # -r: Recursive | -n: Line number | -w: Exact word | -i: Case-insensitive

  # Search for files across the filesystem:
  find / -type f -name "*.conf" 2>/dev/null
  find / -type f -name "*.txt" -user root 2>/dev/null

  # Pentesting Gold Standard: Find SUID binaries (files executing as root):
  find / -perm -4000 -type f 2>/dev/null
  ```

---

### Task 3: Package Management & Tool Installation
* **What it is:** Installing, updating, and removing software on Debian/Ubuntu systems using Advanced Package Tool (`apt`) and `.deb` archives.
* **Why do it:** Target machines often lack offensive/recon tooling (e.g., `netcat`, `curl`, `socat`, or custom dependencies). You must know how to pull dependencies or install offline packages.
* **How to do it:**
  ```bash
  # Update package repositories:
  sudo apt update

  # Install a package:
  sudo apt install -y net-tools curl git

  # Remove a package:
  sudo apt remove --purge nmap

  # Install a local standalone .deb package:
  sudo dpkg -i package.deb
  ```

---

### Task 4: Process Management & Process Forensics
* **What it is:** Inspecting running programs, monitoring CPU/RAM usage, and terminating unresponsive or malicious processes.
* **Why do it:** During VAPT, you inspect running processes to find services running as `root`, identify custom background scripts, or kill antivirus/EDR agents.
* **How to do it:**
  ```bash
  # View full process list with user execution context:
  ps aux
  # a: All users | u: User-oriented format | x: Processes without controlling TTY

  # Filter for a specific service:
  ps aux | grep -i "apache"

  # Real-time resource monitoring:
  top
  htop

  # Terminate processes:
  kill <PID>            # Graceful termination (SIGTERM - 15)
  kill -9 <PID>         # Force kill (SIGKILL - 9)
  killall python3       # Kill all processes by name
  ```

---

### Task 5: Backgrounding & Foregrounding Jobs
* **What it is:** Controlling terminal execution states without terminating processes.
* **Why do it:** Running long-duration tasks (like port scanning, directory fuzzing, or hosting a local web server) in the foreground locks your terminal. Backgrounding frees your shell for continued operations.
* **How to do it:**
  ```bash
  # Start a process directly in the background:
  python3 -m http.server 8000 &

  # Background an active foreground job:
  # 1. Press Ctrl + Z (Suspends process with SIGTSTP)
  # 2. Type 'bg' to send it to the background:
  bg

  # View all background jobs in current session:
  jobs

  # Bring background job back to foreground:
  fg %1
  ```

---

### Task 6: System Service Management (systemd)
* **What it is:** Managing persistent daemon services across boot cycles using `systemctl`.
* **Why do it:** Ensures target services (SSH, web servers, database engines) are active, or stops defensive logging services.
* **How to do it:**
  ```bash
  # Check service status:
  systemctl status ssh
  systemctl status apache2

  # Start, stop, and restart services:
  sudo systemctl start apache2
  sudo systemctl stop apache2
  sudo systemctl restart apache2

  # Enable/disable automatic startup on system boot:
  sudo systemctl enable ssh
  sudo systemctl disable ssh
  ```

---

### Task 7: Automation & Task Scheduling (Cron)
* **What it is:** Scheduling recurring automated scripts or commands using the Cron daemon.
* **Why do it (High-Impact VAPT Focus):**
  1. Cron jobs executing as `root` that call world-writable scripts or relative paths are one of the most common vectors for **Linux Privilege Escalation**.
  2. Attackers install cron jobs to maintain persistent backdoor access across reboots.
* **Structure of a Crontab entry:**
  ```text
  * * * * * command_to_execute
  ┬ ┬ ┬ ┬ ┬
  │ │ │ │ └──── Day of the week (0 - 6, Sunday = 0)
  │ │ │ └────── Month (1 - 12)
  │ │ └──────── Day of the month (1 - 31)
  │ └────────── Hour (0 - 23)
  └──────────── Minute (0 - 59)
  ```
* **How to do it:**
  ```bash
  # View current user's crontab:
  crontab -l

  # Edit current user's crontab:
  crontab -e

  # Inspect system-wide cron jobs (PrivEsc check):
  cat /etc/crontab
  ls -la /etc/cron.*
  ```

---

### Task 8: Security Log Analysis & Forensics
* **What it is:** Inspecting authentication, kernel, and application log files located in `/var/log`.
* **Why do it:** Incident responders inspect logs to detect unauthorized access, while penetration testers review logs to ensure their activities leave minimal signatures or to debug failed exploit attempts.
* **How to do it:**
  ```bash
  # Inspect authentication logs (SSH logins, sudo usage, failed passwords):
  cat /var/log/auth.log         # Debian/Ubuntu
  cat /var/log/secure           # RHEL/CentOS

  # Inspect general system messages:
  cat /var/log/syslog

  # Inspect Apache/Nginx web server access & errors:
  tail -n 50 /var/log/apache2/access.log
  tail -n 50 /var/log/apache2/error.log

  # Filter logs for failed login attempts:
  grep "Failed password" /var/log/auth.log
  ```

---

## 3. High-Impact Linux Interview & PrivEsc Reference

| Technical Concept | Command / Syntax | Pentest / VAPT Relevance |
| :--- | :--- | :--- |
| **SUID Bit** | `find / -perm -4000 2>/dev/null` | Binary executes with owner (root) privileges; exploitable if binary is listed on GTFOBins. |
| **Sudo Rights** | `sudo -l` | Lists commands current user can execute as root without password. |
| **Cron Abuse** | `cat /etc/crontab` | Writable cron scripts running as root allow instant privilege escalation. |
| **Log Evasion** | `history -c && unset HISTFILE` | Clears bash command history to reduce post-exploitation footprint. |
| **Writable Paths** | `find / -writable -type d 2>/dev/null` | Identifies staging directories (`/tmp`, `/dev/shm`) to upload payloads. |

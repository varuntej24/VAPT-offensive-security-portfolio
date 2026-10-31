# Protocols and Servers 2: Attacks on Cleartext Traffic & Mitigation

## Overview
Protocols and Servers 2 focuses on offensive attacks targeting unencrypted legacy services, eavesdropping on network traffic, intercepting active sessions, brute-forcing weak credentials, and implementing defensive cryptographic controls via SSH and SSL/TLS.

---

## Task 1: Introduction
Legacy application layer protocols (HTTP, Telnet, FTP, basic SMTP/POP3/IMAP) transmit commands, data payloads, and authentication credentials across the wire in plaintext. This room explores how adversaries exploit cleartext traffic through sniffing and Man-in-the-Middle (MITM) attacks, performs password auditing, and applies mitigation using Transport Layer Security (TLS) and Secure Shell (SSH).

---

## Task 2: Sniffing Attack

### Mechanics
A packet sniffing attack passively captures network frames as they traverse a network segment. If an application protocol does not utilize cryptographic encapsulation, sensitive data (usernames, session cookies, passwords, private files) is exposed directly within packet payloads.

* **Promiscuous Mode:** Network interface controllers (NICs) normally drop packets not addressed to their MAC address. Sniffing requires setting the NIC to promiscuous mode to capture all broadcast, multicast, and unicast frames on the shared segment.
* **Network Medium Impact:**
  * **Hubs (Legacy):** All frames are broadcast to every port; sniffing is entirely passive.
  * **Switches:** Direct traffic based on CAM/MAC tables; capturing packets intended for other hosts requires active Layer 2 manipulation (e.g., ARP poisoning/spoofing).

### Core Sniffing Utilities
* **CLI Inspection:** `tcpdump`, `tshark`
* **GUI Analysis:** `Wireshark`

```bash
# Capture cleartext HTTP or FTP traffic on interface eth0
sudo tcpdump -i eth0 -A 'tcp port 80 or tcp port 21'

# Inspect cleartext authentication strings directly
sudo tcpdump -i eth0 -A 'tcp port 21' | grep -E "USER|PASS"
```

---

## Task 3: Man-in-the-Middle (MITM) Attack

### Mechanics
In a switched network, an attacker cannot passively sniff traffic between two other nodes without actively altering the network path. A Man-in-the-Middle (MITM) attack forces traffic between a client and a server/gateway through the attacker's machine.

```text
Client (Victim) ───[ Manipulated Route ]───► Attacker Machine ───► Default Gateway / Server
Client (Victim) ◄───[ Read / Alter Data ]─── Attacker Machine ◄─── Default Gateway / Server
```

### Layer 2 MITM: ARP Spoofing / Poisoning
Address Resolution Protocol (ARP) lacks authentication. Any host can send unsolicited ARP replies (Gratuitous ARP) to map an IP address to an arbitrary MAC address.

1. **Poisoning the Client:** Attacker sends forged ARP packets telling the client: *"I have the Gateway's IP address (mapped to Attacker MAC)."*
2. **Poisoning the Gateway:** Attacker sends forged ARP packets telling the gateway: *"I have the Client's IP address (mapped to Attacker MAC)."*
3. **Packet Forwarding:** Attacker enables IP forwarding (`sysctl -w net.ipv4.ip_forward=1`) so packets pass through seamlessly without causing a denial of service, while inspecting or altering payloads in transit.

### Tools Used
* `arpspoof`, `Ettercap`, `Bettercap`

---

## Task 4: Transport Layer Security (TLS)

### Mitigating Cleartext Vulnerabilities
Transport Layer Security (TLS) provides cryptographic confidentiality, integrity, and endpoint authentication between communicating applications, securing protocols like HTTP (into HTTPS), SMTP (STARTTLS), and IMAP/POP3 (IMAPS/POP3S).

```text
[ Application Layer: HTTP / SMTP / IMAP ]
                    │
[ Transport Layer Security: TLS (Symmetric Encryption & HMAC) ]
                    │
[ Transport Layer: TCP (Port 443, 465, 993, etc.) ]
```

### Core Security Properties
1. **Confidentiality:** Symmetrical ciphers (e.g., AES-GCM, ChaCha20) encrypt transmitted data, neutralizing sniffing attacks.
2. **Integrity:** Message Authentication Codes (MAC / HMAC) detect any packet tampering or injection occurring during transit.
3. **Authentication:** X.509 digital certificates validate the server's identity via a trusted public Certificate Authority (CA) chain, preventing basic MITM spoofing.

### Inspection via CLI
```bash
# Verify TLS handshake and inspect certificate chain
openssl s_client -connect <TARGET_IP>:443

# Check for weak or deprecated SSL/TLS cipher suites
nmap --script ssl-enum-ciphers -p 443 <TARGET_IP>
```

---

## Task 5: Secure Shell (SSH)

### Mechanics & Architecture
SSH operates over **TCP Port 22** to replace insecure remote administration tools like Telnet, rlogin, and rsh.

* **Key Exchange:** Uses asymmetric algorithms (e.g., Diffie-Hellman) to establish an ephemeral shared secret without sending the secret over the network.
* **Symmetric Session Encryption:** Derives shared symmetric keys to encrypt all terminal keystrokes, output, and file transfers (SFTP/SCP).

### Authentication Methods
1. **Password Authentication:** Authenticates against system user databases over the encrypted tunnel.
2. **Public Key Authentication:** Cryptographically strong authentication utilizing asymmetric key pairs:
   * **Private Key (`id_rsa` / `id_ed25519`):** Resides locally on the client machine; permissions must be locked down (`chmod 600 id_rsa`).
   * **Public Key (`id_rsa.pub`):** Appended to `~/.ssh/authorized_keys` on the remote target host.

```bash
# Standard SSH connection
ssh username@<TARGET_IP>

# Authenticate using private key
ssh -i id_rsa username@<TARGET_IP>

# Enforce secure permissions on local private keys
chmod 600 id_rsa
```

---

## Task 6: Password Attack

When services (such as SSH, FTP, or web logins) are protected by authentication mechanisms, adversaries attempt unauthorized access via password attacks.

### Attack Classifications
1. **Dictionary Attack:** Iterates through curated wordlists of common passwords (e.g., `rockyou.txt`).
2. **Brute-Force Attack:** Systematically attempts every possible permutation of characters (computationally intensive, slow over networks).
3. **Credential Stuffing:** Replaying leaked username/password combinations harvested from previous data breaches against unrelated services.

### Network Authentication Auditing with Hydra
`hydra` is a multi-threaded network login cracker supporting numerous protocols (SSH, FTP, Telnet, HTTP-POST-Form, SMB, etc.).

```bash
# Dictionary attack against SSH service
hydra -l <USERNAME> -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>

# Dictionary attack against FTP with a user list and password list
hydra -L users.txt -P passwords.txt ftp://<TARGET_IP>

# Fast attack targeting standard HTTP basic authentication
hydra -l admin -P rockyou.txt <TARGET_IP> http-get /admin
```

### Defensive Countermeasures
* Enforce strong password complexity and length requirements.
* Implement account lockout policies or progressive delays (e.g., `fail2ban`).
* Mandate Multi-Factor Authentication (MFA/2FA).
* Disable password-based authentication for remote access in favor of SSH public keys.

---

## Task 7: Summary & Comparison

| Attack Type | Target Layer | Mechanism | Primary Mitigation |
| :--- | :---: | :--- | :--- |
| **Sniffing** | Layer 2 / Layer 3 | Capturing unencrypted network packets | Use TLS / SSH; encrypt all data in transit. |
| **MITM (ARP Spoofing)** | Layer 2 (Data Link) | Poisoning ARP caches to redirect traffic | Dynamic ARP Inspection (DAI), static ARP tables, 802.1X. |
| **Cleartext Interception** | Layer 7 (Application) | Reading plain protocols (HTTP, Telnet, FTP) | Migrate to HTTPS, SSH, SFTP, FTPS. |
| **Password Brute-Force** | Layer 7 (Application) | Automated credential guessing via dictionaries | Key-based auth, MFA, rate-limiting, Fail2ban. |

# Network Protocols & Architecture Cheat Sheet (Day 1)

## 1. The OSI vs. TCP/IP Models
- **Layer 7 - Application:** HTTP/S, DNS, SSH, FTP, SMB (Application interfaces).
- **Layer 4 - Transport:** TCP (reliable, stateful, connection-oriented) and UDP (stateless, connectionless).
- **Layer 3 - Network:** IP addressing, routing, ICMP packets.
- **Layer 2 - Data Link:** MAC addressing, Ethernet frames, ARP resolution.
- **Layer 1 - Physical:** Raw electrical/optical bitstream signals.

---

## 2. TCP 3-Way Handshake & Teardown
- **Establishment:**
  1. `Client` -> `SYN` (Sequence = X) -> `Server`
  2. `Server` -> `SYN-ACK` (Sequence = Y, Acknowledgment = X+1) -> `Client`
  3. `Client` -> `ACK` (Acknowledgment = Y+1) -> `Server`
- **Termination:**
  - `FIN` -> Graceful teardown initiated by either end.
  - `RST` -> Immediate connection reset (port closed, packet dropped, or abnormal termination).

---

## 3. Core Penetration Testing Port Directory
| Port | Protocol | Default Service | Core Security Risk / Vectors |
| :--- | :--- | :--- | :--- |
| **21** | TCP | FTP | Anonymous login, cleartext credentials, exploit vulnerable daemons (e.g., vsftpd 2.3.4). |
| **22** | TCP | SSH | Brute-force attacks, exposed private keys, outdated SSH versions. |
| **53** | TCP/UDP | DNS | DNS zone transfers (`AXFR`), cache poisoning, subdomain enumeration. |
| **80** | TCP | HTTP | Web app flaws (OWASP Top 10), cleartext traffic sniffing. |
| **443** | TCP | HTTPS | SSL/TLS misconfigurations, certificate bypasses, web exploits. |
| **445** | TCP | SMB | Null sessions, EternalBlue (MS17-010), unauthenticated shares, credential relaying. |

---

## 4. Key Diagnostic Commands
- **ICMP Echo Probe:** `ping -c 4 <target-ip>`
- **Route Tracing:** `traceroute <target-ip>` (or `tracert <target-ip>` on Windows)
- **DNS Records Query:** `dig <domain> ANY +noall +answer` (or `nslookup -type=any <domain>` on Windows)
- **DNS Zone Transfer Check:** `dig axfr @<nameserver> <domain>`
- **Domain WHOIS Lookup:** `whois <domain>`

  # 01 - Network Fundamentals & Protocol Mechanics (VAPT Reference)

## Overview
This module documents the foundational networking architectures, protocol state machines, and Layer 2–7 mechanics required for network reconnaissance, packet analysis, and infrastructure penetration testing.

---

## 1. Network Addressing & Device Identification

### Logical vs. Physical Addressing
Devices require two distinct identifiers to communicate across local and external networks:

| Feature | IP Address (Logical) | MAC Address (Physical) |
| :--- | :--- | :--- |
| **OSI Layer** | Layer 3 (Network) | Layer 2 (Data Link) |
| **Structure** | **IPv4:** 32-bit (4 octets, e.g., `192.168.1.77`, ~4.29B addresses)<br>**IPv6:** 128-bit hexadecimal (~340 undecillion addresses) | 48-bit / 12-character Hexadecimal (`a4:c3:f0:85:ac:2d`)<br>• First 3 bytes: OUI (Vendor)<br>• Last 3 bytes: Unique NIC ID |
| **Mutability** | Dynamic (assigned via DHCP or statically) | Burnt into NIC hardware (can be spoofed in software) |
| **Scope** | **Private IP:** Local LAN communication<br>**Public IP:** Routable on the global Internet (assigned by ISP) | Strictly local broadcast domain (stripped at each router hop) |

### VAPT Threat Vector: MAC Address Spoofing
* **Vulnerability Mechanics:** Poorly designed access controls (e.g., captive portals on public Wi-Fi or stateless firewall rules) often rely solely on Layer 2 MAC allowlisting to authenticate administrative hosts.
* **Offensive Execution:** An attacker can sniff network traffic to identify an authorized MAC address and overwrite their local NIC identifier to bypass filtering:
  ```bash
  # Inspect current MAC address
  ip link show eth0

  # Spoof MAC address using macchanger
  sudo ip link set dev eth0 down
  sudo macchanger -m a4:c3:f0:85:ac:2d eth0
  sudo ip link set dev eth0 up
  ```
* **Remediation:** Enforce 802.1X Port-Based Network Access Control and cryptographic authentication rather than relying on Layer 2 identifiers.

---

## 2. LAN Topologies, Switching, Routing & Subnetting

### Network Topologies & Hardware
* **Star Topology:** Endpoints connect to a central Layer 2 **Switch**. High scalability and isolation, though the switch acts as a single point of failure.
* **Bus Topology:** Endpoints share a single backbone cable. Prone to collisions, severe bottlenecks, and complete failure if the backbone breaks.
* **Ring Topology:** Unidirectional loop where nodes forward traffic sequentially.
* **Switch (Layer 2 vs. Layer 3):**
  * **Layer 2 Switch:** Uses ASIC hardware to map MAC addresses to physical switch ports, forwarding frames directly to the target host (unlike legacy Hubs which broadcast to all ports).
  * **Layer 3 Switch:** Performs both Layer 2 frame switching and inter-VLAN IP routing.
* **Router (Layer 3):** Connects distinct networks and routes packets across optimal paths using protocols like **OSPF** (Open Shortest Path First) and **RIP** (Routing Information Protocol).

### Subnetting & VLAN Segregation
Subnetting divides a large network into smaller broadcast domains using a 32-bit **Subnet Mask** (e.g., `255.255.255.0` or `/24`).
* **Network Address (`192.168.1.0`):** Identifies the start of the subnet.
* **Host Address (`192.168.1.100`):** Identifies individual endpoints (up to 254 usable hosts in a `/24`).
* **Default Gateway (`192.168.1.1` or `.254`):** The router interface used to exit the local subnet.
* **VLANs (Virtual LANs):** Logically segregates departments (e.g., *Sales* on `192.168.1.0/24` vs. *Accounting* on `192.168.2.0/24`) on the same physical switch so compromises in one segment cannot directly reach sensitive assets.

---

## 3. Core Glue Protocols: ARP & DHCP

### Address Resolution Protocol (ARP)
Bridges Layer 3 (IP) to Layer 2 (MAC) within a local subnet.
1. **ARP Request (Broadcast):** Host sends `"Who has 192.168.1.20? Tell 192.168.1.10"` to `ff:ff:ff:ff:ff:ff`.
2. **ARP Reply (Unicast):** Target host replies directly with its MAC address (`B5:D1:E2:77:AA:45`).
3. **ARP Cache:** The mapping is stored locally (`arp -a` or `ip neigh`).
* **VAPT Threat Vector (ARP Poisoning / MitM):** Because ARP is stateless and unauthenticated, an attacker can send gratuitous ARP replies claiming to be the Default Gateway, intercepting all local traffic.
* **Remediation:** Enable Dynamic ARP Inspection (DAI) and DHCP Snooping on enterprise switches.

### Dynamic Host Configuration Protocol (DHCP)
Automates IP configuration across four broadcast/unicast phases (**DORA**):
```text
Client (0.0.0.0)                          DHCP Server
   |                                           |
   |--- 1. DHCP Discover (Broadcast) --------->|
   |<-- 2. DHCP Offer (IP: 192.168.1.10) ------|
   |--- 3. DHCP Request (Accept Offer) ------->|
   |<-- 4. DHCP ACK (Lease Confirmed) ---------|
```
* **VAPT Threat Vector:** **DHCP Starvation** (exhausting the IP pool with fake requests) and **Rogue DHCP Servers** (handing out attacker-controlled Default Gateways/DNS servers).

---

## 4. The OSI Model & Encapsulation

The **Open Systems Interconnection (OSI)** model standardizes networking across 7 layers:

| Layer | Name | Protocol Data Unit (PDU) | Core Function & Security Context |
| :---: | :--- | :--- | :--- |
| **7** | **Application** | Data | User/software interface (`HTTP`, `HTTPS`, `FTP`, `SMTP`, `DNS`). |
| **6** | **Presentation** | Data | Data formatting, encoding, and TLS/SSL encryption/decryption. |
| **5** | **Session** | Data | Session establishment, checkpointing, and isolation. |
| **4** | **Transport** | Segment (TCP) / Datagram (UDP) | End-to-end delivery, port addressing (`0–65535`), flow control. |
| **3** | **Network** | **Packet** | Logical IP addressing (`Source/Dest IP`), TTL, routing (`OSPF`/`RIP`). |
| **2** | **Data Link** | **Frame** | Physical MAC addressing, NIC transmission, frame formatting. |
| **1** | **Physical** | Bits (`1`s and `0`s) | Electrical, optical, or radio signals across physical media. |

* **Encapsulation:** Wrapping payload data with layer-specific headers from Layer 7 down to Layer 1 (`Data` -> `Segment` -> `Packet` -> `Frame`).
* **Key IP Packet Headers:**
  * **Time to Live (TTL):** Hop counter decremented by 1 at each router to prevent infinite routing loops.
  * **Checksum:** Integrity verification hash to detect packet corruption in transit.

---

## 5. Transport Layer Mechanics: TCP & Core Ports

### TCP (Transmission Control Protocol)
Connection-oriented protocol mapped across the 4-layer TCP/IP suite (*Application, Transport, Internet, Network Interface*).

#### The TCP 3-Way Handshake & ISN Synchronization
```text
Client (ISN = 0)                              Server (ISN = 5000)
   |                                                   |
   |--- 1. SYN (Seq = 0) ----------------------------->|
   |<-- 2. SYN/ACK (Seq = 5000, Ack = 1) --------------|
   |--- 3. ACK (Seq = 1, Ack = 5001) ----------------->|
   |                                                   |
   |<========= [ ESTABLISHED DATA TRANSFER ] =========>|
   |                                                   |
   |--- 4. FIN/ACK (Graceful Teardown) --------------->|
   |<-- 5. RST (Abrupt Abort / Closed Port Response) --|
```

### Critical VAPT Service Ports (Well-Known Range: `0–1024`)
| Port | Protocol | Service Description & VAPT Enumeration Focus |
| :---: | :--- | :--- |
| **21** | **FTP** | File Transfer Protocol (Cleartext auth, anonymous login checks). |
| **22** | **SSH** | Secure Shell (Encrypted CLI management, key/banner auditing). |
| **80** | **HTTP** | Unencrypted web traffic (Web app vulnerabilities, cleartext sniffing). |
| **443** | **HTTPS** | Encrypted web traffic over TLS/SSL. |
| **445** | **SMB** | Server Message Block (Windows file/printer shares, null sessions, EternalBlue). |
| **3389** | **RDP** | Remote Desktop Protocol (Graphical remote access, credential brute-forcing/BlueKeep). |

---

## 6. Perimeter Defense: Port Forwarding, Firewalls & VPNs

### Port Forwarding (NAT Routing)
Maps an external public IP and port to an internal private host and port (e.g., forwarding `82.62.51.70:80` -> `192.168.1.10:80`), exposing internal intranet services to the public Internet while keeping the rest of the LAN isolated.

### Firewalls: Stateful vs. Stateless Inspection
* **Stateful Firewall:** Tracks the complete connection state table (e.g., verifying a valid TCP 3-way handshake before passing `DATA` packets). High security, higher CPU/memory overhead.
* **Stateless Firewall:** Evaluates individual packets in isolation against static Access Control Lists (ACLs) without session awareness. Fast throughput (useful for mitigating volumetric DDoS), but susceptible to spoofed packet injection (e.g., Nmap ACK scans).

### Virtual Private Networks (VPN)
Establishes an encrypted tunnel connecting geographically separate networks or remote hosts securely over untrusted public infrastructure.
* **PPP (Point-to-Point Protocol):** Non-routable authentication and encryption framework using private keys/certificates.
* **PPTP (Point-to-Point Tunneling Protocol):** Encapsulates PPP for routing; easy to deploy but relies on weak/deprecated cryptography.
* **IPSec (Internet Protocol Security):** Encrypts traffic natively at the IP layer; complex configuration but enterprise-grade cryptographic strength.

---

## 7. Quick Terminal Verification Commands
```bash
# View local IP and MAC addresses
ip addr show

# Inspect local ARP cache table
ip neigh show

# View local routing table and Default Gateway
ip route show

# Trace network path using TTL increments
traceroute 8.8.8.8
```

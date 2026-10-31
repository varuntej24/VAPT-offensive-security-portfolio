# 02 - Nmap Basic Port Scans & Transport Layer Mechanics

## Overview
Port scanning maps the listening network services on a target by probing Transport Layer (TCP and UDP) sockets. This assessment documents the 6 Nmap port states, the TCP control flag architecture, scan execution types (TCP Connect, TCP SYN, UDP), and performance tuning controls.

---

## 1. TCP Header Flags & Handshake Architecture

TCP is a connection-oriented protocol that establishes state using a 3-way handshake. Transport-level scanning relies on manipulating the 6 standard TCP header flags:

| Flag | Full Name | Diagnostic & Operational Purpose |
| :--- | :--- | :--- |
| **SYN** | Synchronize | Initiates a connection and synchronizes initial sequence numbers (ISN). |
| **ACK** | Acknowledge | Confirms receipt of transmitted data payload or handshake segments. |
| **RST** | Reset | Immediately aborts or rejects a connection; returned by closed ports or firewalls. |
| **FIN** | Finish | Gracefully notifies the remote socket that the sender has no more data to transmit. |
| **PSH** | Push | Directs the TCP buffer to immediately push data up to the application layer. |
| **URG** | Urgent | Signals that incoming data pointed to by the Urgent Pointer must be processed immediately. |

---

## 2. The 6 Nmap Port States

Nmap categorizes scanned ports into one of six distinct states based on the packet returned (or dropped):

```text
               ┌── SYN-ACK ───────────────────────────► [ OPEN ]
               ├── RST / RST-ACK ─────────────────────► [ CLOSED ]
Probe (SYN) ───┼── No response (Dropped) / ICMP Type 3 ─► [ FILTERED ]
               └── ACK Probe ──► RST ────────────────► [ UNFILTERED ]
```

| Port State | Technical Meaning | Network Implication |
| :--- | :--- | :--- |
| **`open`** | A service is actively accepting connections on the port. | Target for banner grabbing, version detection, and exploitation. |
| **`closed`** | The port is reachable (host is alive), but no service is bound to it. | Responded with an `RST`; confirms the target is not dropping packets. |
| **`filtered`** | Nmap cannot reach the port to determine state. | Probes were silently dropped by a firewall or returned ICMP unreachable. |
| **`unfiltered`** | Port is accessible, but open/closed status is indeterminate. | Occurs during ACK scans (`-sA`); packet bypassed firewall rules. |
| **`open\|filtered`** | Nmap cannot determine whether the port is open or filtered. | Common in UDP or Null/FIN/Xmas scans where open ports produce no reply. |
| **`closed\|filtered`** | Nmap cannot determine whether the port is closed or filtered. | Observed exclusively in advanced IP ID idle scans. |

---

## 3. Core Scan Types: Packet Flow & Mechanics

### A. TCP Connect Scan (`-sT`)
* **Mechanism:** Relies on the underlying operating system's `connect()` system call to complete the full 3-way handshake.
* **Packet Sequence (Open Port):**
  1. Attacker $\rightarrow$ Target: `[SYN]`
  2. Target $\rightarrow$ Attacker: `[SYN, ACK]`
  3. Attacker $\rightarrow$ Target: `[ACK]` *(Handshake established)*
  4. Attacker $\rightarrow$ Target: `[RST, ACK]` *(Graceful teardown)*
* **Privilege:** Does **not** require raw socket privileges; can be executed by non-root users.
* **Downside:** Easily logged by application-level logging daemons because the connection establishes.

```bash
# Run a TCP Connect scan across standard ports
nmap -sT <TARGET_IP>
```

### B. TCP SYN "Stealth / Half-Open" Scan (`-sS`)
* **Mechanism:** Crafts raw packets directly; interrupts the handshake before completion.
* **Packet Sequence (Open Port):**
  1. Attacker $\rightarrow$ Target: `[SYN]`
  2. Target $\rightarrow$ Attacker: `[SYN, ACK]`
  3. Attacker $\rightarrow$ Target: `[RST]` *(Connection torn down immediately)*
* **Packet Sequence (Closed Port):**
  1. Attacker $\rightarrow$ Target: `[SYN]`
  2. Target $\rightarrow$ Attacker: `[RST, ACK]`
* **Privilege:** Requires `sudo` / `root` to craft raw network frames.
* **Advantage:** Nmap default for privileged users; faster, quieter, and rarely logged by unestablished socket loggers.

```bash
# Run default SYN stealth scan with root privileges
sudo nmap -sS <TARGET_IP>
```

### C. UDP Port Scan (`-sU`)
* **Mechanism:** UDP is connectionless; no handshake exists. Nmap sends raw UDP datagrams (or protocol-specific payloads like DNS queries for port 53).
* **Response Logic:**
  * **No Response / Dropped:** Marked as `open|filtered`.
  * **UDP Response:** Marked as `open`.
  * **ICMP Type 3, Code 3 (Port Unreachable):** Marked as `closed`.
* **Downside:** Linux kernels rate-limit ICMP error generation (often 1 packet/second), making full UDP scans slow.

```bash
# Scan common UDP services (DNS, DHCP, SNMP)
sudo nmap -sU -p 53,67,161 <TARGET_IP>
```

---

## 4. Fine-Tuning Scope & Scanning Performance

### Port Selection Switches
```bash
# Scan a custom comma-separated list
nmap -p 21,22,80,443 <TARGET_IP>

# Scan an inclusive numeric range
nmap -p 1-1024 <TARGET_IP>

# Scan all 65,535 TCP ports
nmap -p- <TARGET_IP>

# Scan the top 100 most common ports (Fast scan)
nmap -F <TARGET_IP>

# Scan the top N most frequent ports according to nmap-services
nmap --top-ports 20 <TARGET_IP>

# Scan ports consecutively instead of randomized order
nmap -r <TARGET_IP>
```

### Timing Templates (`-T0` to `-T5`)
Controls probe timeouts, parallelism, and packet intervals:

| Template | Name | Primary Use Case |
| :---: | :--- | :--- |
| **`-T0`** | **Paranoid** | IDS/IPS evasion; probes one port every 5 minutes. |
| **`-T1`** | **Sneaky** | IDS evasion; probes spaced 15 seconds apart. |
| **`-T2`** | **Polite** | Minimizes bandwidth and target machine load; 10x slower than normal. |
| **`-T3`** | **Normal** | Nmap default dynamic timing model. |
| **`-T4`** | **Aggressive** | Industry standard for reliable, high-speed penetration tests and CTFs. |
| **`-T5`** | **Insane** | Sacrifices accuracy; extremely fast, high risk of packet drop. |

### Packet Rate & Parallelism Controls
```bash
# Enforce minimum and maximum packets sent per second
sudo nmap -sS --min-rate 500 --max-rate 1000 <TARGET_IP>

# Adjust parallel probe threads
sudo nmap -sS --min-parallelism 64 --max-parallelism 256 <TARGET_IP>
```

---

## 5. Technical Interview Reference

* **Why does a SYN scan require `root`, while a Connect scan does not?**
  A standard user can invoke the OS network stack via the `connect()` API call, which initiates and completes the TCP handshake automatically. To send an isolated `SYN` and immediately terminate it with a raw `RST` without completing the handshake, Nmap must craft custom raw IP packets using raw sockets (`AF_INET`, `SOCK_RAW`), an operation restricted to root/CAP_NET_RAW capabilities.
* **Why does a closed port return `RST` instead of nothing?**
  Per RFC 793, if a connection request arrives at an accessible host on a port with no listening process, the TCP implementation must generate an `RST` segment to inform the client not to retry.
* **Why do UDP scans frequently show `open|filtered`?**
  UDP provides no acknowledgment mechanism. If an open service does not send application-layer data back, Nmap cannot distinguish whether the probe was received by an open service or silently dropped by a stateful firewall.

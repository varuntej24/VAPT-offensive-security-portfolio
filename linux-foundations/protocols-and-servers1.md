# 04 - Application Layer Protocols and Servers

## Overview
This module explores common unencrypted Application Layer protocols (OSI Layer 7 / TCP/IP Layer 4). Because these legacy protocols transmit both commands and credentials in cleartext, standard network clients or raw terminal sockets (`telnet`, `nc`) can interact directly with the daemons. 

This assessment breaks down:
1. Remote administration via Telnet.
2. Web delivery via HTTP.
3. File transfer architecture (Active vs. Passive) via FTP.
4. Internet mail infrastructure (SMTP, POP3, IMAP).
5. Vulnerabilities inherent to cleartext protocols.

---

## 1. Protocol Architecture & Default Ports

| Protocol | Default Port | Transport | Purpose | Secure Equivalent (Port) |
| :--- | :---: | :---: | :--- | :--- |
| **Telnet** | `23` | TCP | Unencrypted remote terminal session | **SSH** (`22`) |
| **HTTP** | `80` | TCP | Web resource retrieval & hypermedia | **HTTPS** (`443`) |
| **FTP** (Control) | `21` | TCP | File transfer commands and auth | **SFTP** (`22`) / **FTPS** (`990`) |
| **FTP** (Data) | `20` | TCP | File payload transmission (Active Mode) | — |
| **SMTP** | `25` | TCP | Mail routing / transfer between MTAs | **SMTPS** (`465`) / Submission (`587`) |
| **POP3** | `110` | TCP | Mail retrieval from MDA (Download & Delete) | **POP3S** (`995`) |
| **IMAP** | `143` | TCP | Mail retrieval with server synchronization | **IMAPS** (`993`) |

---

## 2. Remote Shell Management: Telnet (Port 23)

Telnet provides a bidirectional, interactive text-oriented communication channel over TCP.

* **Mechanics:** Once connected, the server prompts for authentication (`login:`, `password:`). If authenticated, it returns an interactive shell prompt (e.g., `user@host:~$`).
* **Security Risk:** Everything (including login credentials and executed shell commands) passes across the wire without encryption, enabling trivial credential sniffing via passive packet capture.

```bash
# Connect to a remote Telnet service
telnet <TARGET_IP> 23
```

---

## 3. Web Delivery: HTTP (Port 80)

HTTP delivers web assets using a client-request/server-response model. Instead of relying on a graphical web browser, raw HTTP requests can be crafted and transmitted directly via a Telnet or Netcat socket.

### Manual HTTP Interaction
Connecting to port 80 and sending a minimal HTTP/1.1 request requires specifying the target resource and the mandatory `Host` header, followed by a double newline:

```text
$ telnet <TARGET_IP> 80
Trying <TARGET_IP>...
Connected to <TARGET_IP>.
Escape character is '^]'.

GET /index.html HTTP/1.1
Host: webserver

HTTP/1.1 200 OK
Date: Sat, 31 Oct 2026 12:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html
Content-Length: 142

<html>
  <body>Welcome to the Target Server</body>
</html>
```

* **Common Web Servers:** Apache (`httpd`), Nginx, Microsoft Internet Information Services (IIS).

---

## 4. File Transfers: FTP (Port 21)

File Transfer Protocol (FTP) establishes **two separate TCP connections** to handle operations:
1. **Control Connection (Port 21):** Carries authentication, directory traversal commands, and status responses.
2. **Data Connection:** Carries actual file listings (`ls`) and file uploads/downloads (`get`, `put`).

### Active vs. Passive FTP Modes

```text
Active Mode:
Client (Port >1023) ──────── Connect Control (Port 21) ────────> Server
Client (Port >1023) <─────── Server Opens Data (Port 20) ─────── Server

Passive Mode (PASV):
Client (Port >1023) ──────── Connect Control (Port 21) ────────> Server
Client (Port >1023) ──────── Connect Data (Port >1023) ────────> Server
```

* **Active Mode:** The client issues `PORT <IP,port>` over the control channel. The FTP server initiates the data connection from its own **port 20** back to the client's high port. Modern client-side firewalls frequently block this inbound connection.
* **Passive Mode (`PASV`):** The client issues the `PASV` command. The server opens an unassigned high port (above 1023) and tells the client to initiate the inbound data connection, bypassing client-side NAT/firewalls.

### Manual FTP Protocol Commands (via Telnet)
```text
$ telnet <TARGET_IP> 21
220 (vsFTPd 3.0.3)
USER frank
331 Please specify the password.
PASS D2xc9CgD
230 Login successful.
SYST
215 UNIX Type: L8
PASV
227 Entering Passive Mode (10,10,113,62,195,12).
QUIT
221 Goodbye.
```

---

## 5. Internet Mail Infrastructure: SMTP, POP3, and IMAP

Email architecture is divided into two primary functions: **Mail Transfer/Sending** and **Mail Access/Retrieval**.

```text
Sender (MUA) ──SMTP──> Sending Server (MSA/MTA) ──SMTP──> Receiving Server (MTA/MDA)
                                                                 │
                                                            POP3 or IMAP
                                                                 │
                                                                 ▼
                                                        Recipient (MUA)
```

* **MUA (Mail User Agent):** The client program (Thunderbird, Outlook, or webmail interface).
* **MSA (Mail Submission Agent):** Receives email from the sending MUA and passes it to the MTA.
* **MTA (Mail Transfer Agent):** Routes mail across internet relays using DNS MX records (e.g., Postfix, Sendmail, Exim).
* **MDA (Mail Delivery Agent):** Stores incoming messages into local recipient mailboxes.

---

### A. Simple Mail Transfer Protocol (SMTP - Port 25)
Used strictly for **pushing/routing** messages to MTAs.

```text
$ telnet <TARGET_IP> 25
220 mail.local ESMTP Postfix
HELO attacker.local
250 mail.local
MAIL FROM: <alice@target.com>
250 2.1.0 Ok
RCPT TO: <bob@target.com>
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Subject: Test Email

This is a test message.
.
250 2.0.0 Ok: queued as 4SKL20934
QUIT
221 2.0.0 Bye
```

---

### B. Post Office Protocol 3 (POP3 - Port 110)
A simple retrieval protocol. By default, it connects, downloads the inbox messages to the local client, and flags/deletes them from the server.

* **Limitation:** Poor for users checking mail across multiple devices (desktops, phones), as emails downloaded to one device disappear from the server.

```text
$ telnet <TARGET_IP> 110
+OK POP3 server ready
USER frank
+OK
PASS D2xc9CgD
+OK Logged in.
STAT
+OK 2 1420
LIST
+OK 2 messages:
1 500
2 920
.
RETR 1
+OK 500 octets
<Message 1 Content>
.
QUIT
+OK Bye.
```

---

### C. Internet Message Access Protocol (IMAP - Port 143)
An advanced retrieval protocol designed for **bi-directional synchronization**. Messages, folders, flags (read, replied, flagged), and drafts remain stored on the central MDA server, keeping all client devices in sync.

* **Syntax Note:** IMAP requires each command to be prepended with a unique client-generated tag/string (e.g., `a1`, `c1`) so asynchronous server responses can be correlated with their requests.

```text
$ telnet <TARGET_IP> 143
* OK IMAP4rev1 Server Ready
c1 LOGIN frank D2xc9CgD
c1 OK LOGIN completed
c2 LIST "" "*"
* LIST (\HasNoChildren) "/" "INBOX"
c2 OK LIST completed
c3 SELECT INBOX
* 2 EXISTS
c3 OK [READ-WRITE] Select completed
c4 LOGOUT
* BYE IMAP4rev1 Server logging out
c4 OK LOGOUT completed
```

---

## 6. Penetration Testing & VAPT Takeaways

1. **Cleartext Exploitation:**
   Every protocol examined in this module (Telnet, HTTP, FTP, SMTP, POP3, IMAP) lacks native transport-layer encryption. Any adversary with layer-2 access (ARP spoofing) or an upstream compromise can harvest cleartext credentials using packet analyzers like Wireshark or `tcpdump`.
2. **Raw Socket Diagnostic Capability:**
   When testing firewalls, web proxies, or broken services, never rely solely on browsers or heavy tools. Connecting via `telnet <IP> <PORT>` or `nc -vn <IP> <PORT>` allows you to bypass client-side validation and directly observe raw daemon banner messages and protocol errors.
3. **Hardening & Migration:**
   * Replace Telnet with **SSH**.
   * Enforce **HTTPS** with valid TLS certificates.
   * Require **FTPS** or move transfers entirely to **SFTP**.
   * Require TLS encryption wrappers for mail transmission: **SMTPS** (Port 465) / **STARTTLS** (Port 587) for sending, and **POP3S** (Port 995) or **IMAPS** (Port 993) for inbox retrieval.

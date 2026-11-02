# Ethical Hacking Lifecycle & Web Authentication Bypass Mechanics

## Overview
The **Become a Hacker** module explores the operational methodology of ethical hacking versus malicious exploitation. It focuses on the offensive reconnaissance cycle: locating hidden entry points, assessing authentication endpoints, executing password-spraying/brute-force attacks, and post-exploitation validation under strict rules of engagement.

---

## 1. Attack Surface Discovery & Endpoint Probing
Before launching intrusive attacks, an ethical hacker maps out the target web application to identify unlinked or administrative surfaces.

* **Target:** `http://www.onlineshop.thm`
* **Discovery Vector:** URL path manipulation and directory brute-forcing.
* **Finding:** While typical users only see public storefront pages (`/products`, `/about`), guessing or fuzzing directory endpoints reveals administrative access points:
  ```http
  GET /login HTTP/1.1
  Host: www.onlineshop.thm
  ```
* **Security Risk:** Publicly exposing internal or administrative login portals without IP whitelisting or network isolation increases the risk of credential brute-force attacks.

---

## 2. Authentication Brute-Force Mechanics (Hydra)
When an authentication form lacks rate-limiting, account lockout mechanisms, or CAPTCHA defenses, attackers can systematically test high-probability credentials.

### Command Execution:
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt www.onlineshop.thm http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

### Parameter Breakdown:
* `-l admin`: Specifies a single static target username.
* `-P <wordlist>`: Defines the password wordlist dictionary to test.
* `http-post-form`: Dictates the protocol module and specifies the target path, variable parameters (`^USER^`, `^PASS^`), and failure response string (`Invalid credentials`) used to confirm an incorrect attempt.

### Post-Exploitation & Validation:
* **Discovered Credentials:** `admin:qwerty`
* **Exploitation Impact:** Access to the backend administrative console allowed access to sensitive flags (`born_to_be_a_hacker`), simulating complete administrative takeover.

---

## 3. Defense & Hardening Recommendations

| Weakness | Attack Vector | Remediation |
| :--- | :--- | :--- |
| **Exposed Administrative Portal** | Endpoint Enumeration | Restrict admin portals to internal VPNs or implement IP allowlisting. |
| **Weak Credentials** | Dictionary Attacks | Enforce password complexity policies prohibiting default and known dictionary passwords. |
| **No Rate Limiting** | Automated Credential Spraying | Implement exponential backoff, account lockouts after 5 failed attempts, and multi-factor authentication (MFA). |

# Burp Suite: Intruder Mechanics & Automated Fuzzing

## Overview
Burp Suite Intruder is an automated request fuzzing and credential testing engine. While **Repeater** handles isolated, manual request/response validation, **Intruder** automates high-volume request streams with customized payload sets, dynamic variable insertion, and result extraction rules.

This module covers the core mechanics of Intruder:
1. **Targeting & Payload Positions:** Defining injection delimiters (`§`) and matching attack types to exploitation goals.
2. **The 4 Attack Types:** Operational differences between Sniper, Battering Ram, Pitchfork, and Cluster Bomb.
3. **Payload Processing & Encodings:** Transforming payloads on the fly (Base64, URL encoding, prefix/suffix additions).
4. **Result Triage & Extraction:** Using Status Codes, Response Lengths, Grep - Match, and Grep - Extract to identify valid responses.

---

## 1. Attack Types & Payload Positioning

Intruder marks injection points using section symbols (`§value§`). The attack type dictates how multiple payload sets map to those defined positions.

```text
Target Request Snippet:
POST /login HTTP/1.1
Host: target.thm
...
username=§admin§&password=§password123§
```

| Attack Type | Payload Sets Required | Position Execution Behavior | Primary Security Use Case |
| :--- | :--- | :--- | :--- |
| **Sniper** | 1 Set | Tests **one position at a time**. Cycles the payload list through Position 1 while keeping Position 2 static; then tests Position 2 while keeping Position 1 static. | Parameter fuzzing, single-credential spraying, SQLi/XSS reflection probing. |
| **Battering Ram** | 1 Set | Injects the **same payload simultaneously** into all marked positions on each request. | Testing systems where username must match password, or duplicate parameters (`token=X&confirm_token=X`). |
| **Pitchfork** | Multiple Sets (1 per position) | Iterates through multiple payload lists **in parallel lockstep** (Row 1 with Row 1, Row 2 with Row 2). Stops when the shortest list runs out. | Credential pair stuffing from leaked breach dumps (`user:pass` pairs), multi-field token testing. |
| **Cluster Bomb** | Multiple Sets (1 per position) | Tests **every possible permutation** ($N \times M$ matrix). Tests Payload 1 from Set A against every payload in Set B before advancing. | Full credential brute-forcing (unknown username list $\times$ wordlist passwords). |

---

## 2. Payload Types & In-Flight Processing

Intruder supports multiple native generators to craft payloads without external scripts:

* **Simple List:** Static wordlists loaded from file or SecLists (e.g., `rockyou.txt`, `common-passwords.txt`).
* **Runtime File:** Reads line-by-line from disk during the attack to minimize Burp memory overhead on large dictionaries.
* **Numbers / Dates:** Sequential or stepped integers (useful for IDOR parameter brute-forcing like `1` to `1000`).
* **Brute Forcer:** Generates character-set combinations of a defined length.

### Payload Processing Rules
Rules execute sequentially before a payload is placed into the request wire:
1. **Prefix / Suffix:** Appending static strings (e.g., prepending `Bearer ` to authentication tokens).
2. **Match / Replace:** Regex replacement inside wordlist items.
3. **Encode / Decode:** Base64, URL, Hex, or Hash generation (MD5, SHA-256).
4. **URL-Encode Characters:** Automatically encodes characters (`=`, `&`, spaces) to avoid breaking raw HTTP query syntax.

---

## 3. Response Analysis & Result Triage

On high-volume attacks, manual inspection of every response is impossible. Triage relies on automated filtering indicators:

* **HTTP Status Code:** `200 OK` vs. `302 Found` (redirect to dashboard) vs. `401 Unauthorized` or `403 Forbidden`.
* **Response Length:** When an application returns `200 OK` for both failed and successful attempts, length differentials reveal success (e.g., failed login = `3420 bytes`, successful login = `4180 bytes`).
* **Grep - Match:** Searches the response body for specific success or failure strings (e.g., flag responses that DO NOT contain `"Invalid username or password"`, or DO contain `"Welcome, admin"`).
* **Grep - Extract:** Automatically parses and extracts specific text blocks (e.g., CSRF anti-automation tokens, session identifiers, internal error codes) directly into the Intruder results table.

---

## 4. Technical Interview Defense Points

* **"What is the operational difference between Pitchfork and Cluster Bomb?"**
  Pitchfork iterates through multiple lists in parallel synchronization—Request 1 pairs `user[0]` with `pass[0]`, Request 2 pairs `user[1]` with `pass[1]`. Total requests equal the length of the shortest list ($N$). Cluster Bomb creates a cartesian product of all lists—every username is tested against every password ($N \times M$). Pitchfork is used for testing known correlated pairs, while Cluster Bomb is used for exhaustive unknown credential combinations.
* **"Why is Burp Suite Community Intruder rarely used for large-scale directory or password brute-forcing in real assessments?"**
  Burp Suite Community edition enforces deliberate algorithmic throttling on Intruder, reducing request throughput to ~1 request per second. For extensive wordlists (SecLists, RockYou), penetration testers use dedicated CLI tools like **`ffuf`** or **`hydra`**, reserving Burp Intruder for targeted parameter fuzzing, proof-of-concept validation, and small payload checks.

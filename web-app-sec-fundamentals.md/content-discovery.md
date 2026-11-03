# Web Application Content Discovery & Reconnaissance

## 1. Manual Reconnaissance & Information Disclosure
* **Robots & Sitemaps:** Audited `/robots.txt` and `/sitemap.xml` to uncover hidden paths, staging endpoints, and administrative interfaces exposed through crawler directives.
* **Header Inspection:** Examined HTTP response headers (`Server`, `X-Powered-By`, framework cookies) to identify underlying infrastructure, operating systems, and backend runtimes.
* **Favicon Fingerprinting:** Generated cryptographic hashes (MD5 / MurmurHash3) of `favicon.ico` to cross-reference against known framework signatures (Spring Boot, Drupal, Jenkins).

## 2. OSINT & Passive Discovery
* **Search Engine Operators (Google Dorking):** Utilized advanced filters (`site:`, `filetype:`, `inurl:`, `intitle:`) to locate cached documents, login portals, and leaked database files.
* **Web Archives:** Queried the Wayback Machine to trace legacy endpoints, historical parameters, and deprecated API versions (`/api/v1/`).
* **Source Repositories:** Scanned public GitHub/GitLab repositories for exposed secrets, internal staging domains, and configuration leaks.

## 3. Automated Discovery & Gobuster
* **Directory Enumeration:** Executed dictionary-based brute forcing against target web roots to identify unlinked files and directories.
  ```bash
  gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
  ```
* **DNS Subdomain Enumeration:** Discovered external network hosts by querying DNS records across targeted wordlists.
  ```bash
  gobuster dns -d target.thm -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
  ```
* **Virtual Host (`vhost`) Discovery:** Fuzzed the HTTP `Host` header to uncover internal virtual hosts routed across the same IP address without dedicated DNS entries.
  ```bash
  gobuster vhost -u http://<TARGET_IP> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
  ```

## 4. Remediation & Hardening Guidelines
1. **Restrict Sensitive Directives:** Remove sensitive administrative paths from public `robots.txt` files; enforce access control at the authentication layer instead.
2. **Scrub Deployment Artifacts:** Remove version control directories (`.git`), temporary editor backups (`.bak`, `.swp`), and local environment configurations (`.env`) from production builds.
3. **Disable Directory Indexing:** Turn off directory browsing (`Options -Indexes` in Apache, `autoindex off` in Nginx) to eliminate recursive directory traversal.

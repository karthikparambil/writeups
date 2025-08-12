# Ignite CTF – TryHackMe Write-Up

**Room:** Ignite  
**Difficulty:** Easy  
**Author:** Karthik

---

##  Overview  
Ignite presents a realistic scenario: a fledgling start-up running an outdated CMS on their web server. Your mission—root the box—teaches valuable lessons in reconnaissance, exploitation, and privilege escalation.

---

## 1. Reconnaissance & Enumeration

- **Port Scanning:** Only port **80 (HTTP)** is open—no other services exposed. :contentReference[oaicite:1]{index=1}  
- **robots.txt:** Reveals a disallowed path: `/fuel`, pointing to Fuel CMS. :contentReference[oaicite:2]{index=2}  
- **Fuel CMS Identification:** The CMS running is **Fuel CMS 1.4 / 1.4.1**, a version known to have RCE vulnerabilities. :contentReference[oaicite:3]{index=3}  

---

## 2. Gaining Foothold (RCE Exploitation)

- The `admin:admin` default credentials often work for login. :contentReference[oaicite:4]{index=4}  
- However, exploitation doesn’t strictly require authentication—Fuel CMS 1.4.1 is vulnerable (CVE-2018-16763) via the `filter` parameter at `/fuel/pages/select`, allowing **Remote Code Execution**. :contentReference[oaicite:5]{index=5}  
- Using a Python exploit script (e.g., `50477.py`), RCE is triggered and a shell command interface is obtained. :contentReference[oaicite:6]{index=6}  
- **Reverse Shell:** Deploying a netcat-based reverse shell (via `mkfifo`) gives an interactive shell as `www-data`. :contentReference[oaicite:7]{index=7}  

---

## 3. Capturing the User Flag

Once shell access as `www-data` is established, retrieving the user flag is straightforward:

```bash
cat /home/www-data/user.txt

